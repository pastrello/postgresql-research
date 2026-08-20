# Auditoria inicial — PostgreSQL 18.6

Baseline: branch `upstream-18.6`, commit `76b23567d9e283fc2629374d4a8b2871a023d965`.

Esta auditoria procura oportunidades de ganho operacional mensurável sem assumir que o PostgreSQL upstream esteja "errado". Nenhum item de segurança abaixo representa, por si só, uma vulnerabilidade confirmada; são propostas de hardening para um fork experimental.

## Escopo desta rodada

- heap sequential scans;
- ReadStream / AIO;
- BufferAccessStrategy;
- custo do planner para I/O e paralelismo;
- observabilidade necessária para benchmarks;
- limites de mensagens do protocolo;
- carregamento de bibliotecas dinâmicas.

## Resultado executivo

| ID | Prioridade | Área | Hipótese / oportunidade | Decisão |
|---|---|---|---|---|
| PERF-001 | P0 | I/O / buffers | Ring `BAS_BULKREAD` usa `effective_io_concurrency` global enquanto ReadStream usa valor específico do tablespace | Primeiro experimento |
| OBS-001 | P0 | Observabilidade | ReadStream adapta look-ahead, mas não expõe métricas suficientes para explicar combinação, ramp-up e espera por I/O | Instrumentar antes de alterações agressivas |
| PERF-002 | P1 | Planner | `cost_seqscan()` não modela concorrência AIO; em plano paralelo divide CPU, mas não o custo de disco | Benchmark antes de alterar |
| PERF-003 | P1 | ReadStream | Seq scan usa `READ_STREAM_SEQUENTIAL`, não `READ_STREAM_FULL`; portanto começa com distância 1 e faz ramp-up | Testar em storage de alta latência |
| PERF-004 | P2 | Buffer manager | Ativação de `BAS_BULKREAD` usa limiar fixo `NBuffers / 4` | Tornar ajustável mantendo default |
| PERF-005 | P2 | Paralelismo | Número de workers cresce pela regra geométrica baseada em páginas, sem considerar capacidade real de I/O | Só depois de medir PERF-001/002 |
| SEC-001 | P1 hardening | Protocolo | Mensagens Query/Bind/Parse/CopyData aceitam limite grande próximo a `MaxAllocSize` | GUC opcional, default compatível |
| SEC-002 | P2 hardening | Extensões | Loader dinâmico permite caminhos resolvidos pelo search path quando a chamada não é `restricted` | Allowlist opcional para ambientes fechados |
| BUILD-001 | P0 qualidade | Build/teste | Sanitizers e assert builds devem acompanhar qualquer patch | Implementar no laboratório |

## PERF-001 — inconsistência entre concorrência do tablespace e ring bulk-read

### Evidência no código

1. `src/backend/access/heap/heapam.c::initscan()` ativa `BAS_BULKREAD` para relações maiores que `NBuffers / 4`.
2. `src/backend/storage/buffer/freelist.c::GetAccessStrategy(BAS_BULKREAD)` dimensiona o ring usando `io_combine_limit * effective_io_concurrency`, onde `effective_io_concurrency` é o valor global.
3. `src/backend/storage/aio/read_stream.c::read_stream_begin_impl()` obtém `max_ios` com `get_tablespace_io_concurrency(tablespace_id)`, portanto respeita override do tablespace.
4. O mesmo `read_stream_begin_impl()` reduz `max_pinned_buffers` por `GetAccessStrategyPinLimit(strategy)`. Para `BAS_BULKREAD`, esse limite é o tamanho do ring.

### Hipótese

Se o servidor tiver, por exemplo, `effective_io_concurrency = 1` global e um tablespace rápido configurado com `effective_io_concurrency = 32`, o ReadStream solicitará a concorrência do tablespace, mas a capacidade de manter buffers em voo poderá ser limitada por um ring calculado com o valor global menor.

Isso não prova regressão: o ring tem mínimo de 256 kB e outros limites entram no cálculo. A hipótese precisa ser testada com `io_method=worker` e `io_method=io_uring`, quando disponível.

### Patch preferido

Introduzir cálculo de ring bulk-read consciente do tablespace, evitando mudar defaults quando não houver override. A primeira implementação deve ser pequena e reversível, preferencialmente sem alterar semântica de outros usuários de `GetAccessStrategy()`.

## OBS-001 — métricas do ReadStream

`ReadStream` já possui um algoritmo adaptativo interessante: inicia com distância pequena, aumenta rapidamente após I/O, reduz em cache hits, combina blocos contíguos e controla I/Os em voo. Antes de mexer nesse algoritmo, precisamos enxergar o comportamento real.

Métricas candidatas:

- blocos solicitados ao callback;
- blocos encontrados em cache;
- blocos que exigiram I/O;
- número de operações iniciadas;
- blocos por operação (combine ratio);
- distância inicial, média e pico;
- waits de `WaitReadBuffers()`;
- vezes em que o pin limit reduziu a distância;
- leituras encurtadas/forwarded buffers.

A forma inicial mais segura é instrumentação de desenvolvimento, sem ABI pública. Depois podemos decidir se alguma métrica merece `EXPLAIN` ou `pg_stat_io`.

## PERF-002 — custo de Seq Scan e AIO

`cost_seqscan()` calcula custo de disco como `spc_seq_page_cost * baserel->pages`. Em plano paralelo, o custo de CPU é dividido pelo divisor paralelo, mas o código explicitamente não amortiza o custo de I/O.

Essa política pode continuar correta em muitos ambientes, mas PG18 já possui AIO e ReadStream. Portanto, merece experimento controlado para saber se o planner passa a subestimar alternativas que se beneficiam de I/O concorrente ou se uma correção causaria regressões em storage já bem servido pelo readahead do SO.

Não alterar o planner antes de obter dados de OBS-001 e PERF-001.

## PERF-003 — ramp-up em scans longos

`read_stream_begin_impl()` inicia `distance = 1`, exceto quando recebe `READ_STREAM_FULL`, quando começa já assumindo leituras de até `io_combine_limit`.

Em `heap_beginscan()`, sequential scans usam `READ_STREAM_SEQUENTIAL | READ_STREAM_USE_BATCHING`, mas não `READ_STREAM_FULL`.

A decisão é defensável porque um Seq Scan pode terminar cedo por um nó superior (por exemplo `LIMIT`). O experimento é testar um hint vindo do executor/planner somente quando a fração estimada a consumir for alta. Potencial benefício deve ser pequeno em storage de baixa latência porque o ramp-up dobra rapidamente.

## PERF-004 — limiar fixo para bulk-read

`initscan()` usa estratégia bulk-read e synchronized scan quando a relação ultrapassa `NBuffers / 4` (salvo flags que desabilitem cada comportamento).

O default deve permanecer intacto. Um GUC experimental permitiria descobrir se workloads de pesquisa com `shared_buffers` grande ganham ao ativar o ring mais cedo ou mais tarde.

## PERF-005 — workers paralelos

`compute_parallel_worker()` deriva workers do tamanho da heap/index e multiplica o limiar por 3 a cada worker adicional. O próprio comentário no código reconhece que a heurística poderia ser mais sofisticada.

Não é um bom primeiro patch: alterar workers muda CPU, memória, I/O, Gather e contenção simultaneamente. Só será investigado depois que o comportamento de I/O estiver medido.

## SEC-001 — limite configurável para mensagens grandes

`PQ_LARGE_MESSAGE_LIMIT` é `MaxAllocSize - 1`. `SocketBackend()` usa esse limite para Query, FunctionCall, Bind, Parse e CopyData.

Proposta de hardening: GUC(s) opcional(is) para teto de mensagens de frontend, mantendo como default o comportamento upstream. O objetivo é limitar dano de clientes autenticados defeituosos/comprometidos e reduzir risco de pressão de memória por mensagens gigantes. Deve haver cuidado especial com COPY e clientes legítimos que enviam parâmetros grandes.

## SEC-002 — allowlist de bibliotecas dinâmicas

`load_external_function()` resolve o nome pelo `dynamic_library_path` e chama o loader. O caminho `restricted` já possui regra própria para `$libdir/plugins/`.

Proposta experimental: allowlist opcional de diretórios/nomes permitidos para instalações que desejem reduzir superfície pós-comprometimento. Não tratar isso como correção de vulnerabilidade e não mudar o default upstream.

## BUILD-001 — disciplina de validação

Todo patch funcional deverá passar, no mínimo:

1. build normal otimizado;
2. build com assertions;
3. regression tests aplicáveis;
4. ASan/UBSan quando compatível;
5. benchmark repetido contra o mesmo commit de `upstream-18.6`;
6. teste de regressão específico do patch;
7. registro de resultado em `accepted` ou `rejected`.

## Ordem recomendada

1. OBS-001 — instrumentação mínima de ReadStream;
2. PERF-001 — ring bulk-read consciente do tablespace;
3. benchmark A/B;
4. PERF-002 — só se os dados mostrarem divergência relevante entre custo e execução;
5. PERF-003/004;
6. segurança/hardening em branch separada.
