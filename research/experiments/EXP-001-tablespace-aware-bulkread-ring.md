# EXP-001 — Tablespace-aware BAS_BULKREAD ring

Status: `planned`
Prioridade: P0
Risco esperado: baixo/médio

## Pergunta

Um override de `effective_io_concurrency` no tablespace pode ser limitado indiretamente pelo tamanho do ring `BAS_BULKREAD`, que hoje é calculado com o valor global?

## Caminho de código

- `heapam.c::initscan()` cria `BAS_BULKREAD` em relações grandes.
- `freelist.c::GetAccessStrategy()` dimensiona o ring com `io_combine_limit * effective_io_concurrency` global.
- `read_stream.c::read_stream_begin_impl()` lê `get_tablespace_io_concurrency(tablespace_id)`.
- `read_stream_begin_impl()` limita `max_pinned_buffers` com `GetAccessStrategyPinLimit(strategy)`.
- `GetAccessStrategyPinLimit(BAS_BULKREAD)` retorna o número de buffers do ring.

## Cenário que deve revelar o problema

- relação significativamente maior que `shared_buffers`;
- relação armazenada em tablespace dedicado;
- `effective_io_concurrency` global baixo;
- `effective_io_concurrency` do tablespace significativamente maior;
- AIO habilitado (`worker` e, quando disponível, `io_uring`);
- query de varredura com pouco custo de CPU, para destacar I/O.

Exemplo de matriz:

| Caso | Global | Tablespace | Objetivo |
|---|---:|---:|---|
| A | 1 | 1 | controle |
| B | 1 | 16 | detectar limitação pelo ring |
| C | 1 | 64 | ampliar sinal |
| D | 16 | 16 | comparar com B |
| E | 64 | 64 | referência de ring grande |

## Hipótese de patch

A opção mais conservadora é separar o cálculo do tamanho do ring e permitir que o chamador forneça a concorrência efetiva do tablespace. Evitar alteração global de `GetAccessStrategy()` até sabermos quantos outros callers seriam afetados.

Possíveis formas:

1. novo helper `GetAccessStrategyForTablespace()`;
2. helper que calcula tamanho recomendado e chama `GetAccessStrategyWithSize()`;
3. informação de concorrência passada pelo heap AM no momento de criar o strategy.

A opção escolhida deve manter comportamento bit-a-bit equivalente quando o tablespace não tiver override.

## Métricas

- tempo total / mediana / p95;
- throughput de leitura;
- IOPS;
- CPU user/system;
- `pg_stat_io` antes/depois;
- blocos lidos e hits em `EXPLAIN (ANALYZE, BUFFERS)`;
- número de I/Os e combine ratio quando OBS-001 estiver disponível;
- pico de buffers pinados e vezes em que o pin limit reduziu look-ahead.

## Critério de aceitação

Manter o patch somente se:

- houver ganho repetível no cenário com override de tablespace;
- o cenário sem override permanecer estatisticamente equivalente;
- não houver regressão relevante em `sync` ou em tablespace lento;
- regression tests passarem.
