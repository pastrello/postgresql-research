# Baseline de benchmark — PostgreSQL 18.6

## Objetivo

Garantir que cada patch seja comparado com a mesma baseline, usando dados e configuração reproduzíveis.

## Builds

Manter pelo menos dois builds da mesma revisão:

1. **release-like**: otimizado, usado para números de desempenho;
2. **diagnostic**: assertions + símbolos; ASan/UBSan quando aplicável.

Não comparar um build com assertions/sanitizers contra um build release para decidir desempenho.

## Dataset

Para testes de I/O, criar pelo menos uma relação cuja heap seja várias vezes maior que `shared_buffers`. Idealmente usar um segundo dataset ainda maior para validar escalabilidade.

Registrar:

- tamanho total da relação e índices;
- `shared_buffers`;
- memória RAM do host;
- filesystem;
- dispositivo/storage;
- kernel;
- CPU;
- versão do compilador;
- flags de build.

## Configurações que devem acompanhar cada resultado

- `io_method`;
- `io_workers`;
- `io_max_concurrency`;
- `io_combine_limit`;
- `effective_io_concurrency` global;
- override de `effective_io_concurrency` do tablespace;
- `shared_buffers`;
- `effective_cache_size`;
- `max_parallel_workers_per_gather`;
- `min_parallel_table_scan_size`;
- `seq_page_cost` e `random_page_cost`.

## Workloads iniciais

### W1 — I/O dominante

Uma agregação simples que force leitura da heap e tenha pouco trabalho por tupla, repetida com cache frio e quente.

### W2 — CPU + I/O

Agregação com expressão simples para verificar se o ganho de I/O continua relevante quando existe trabalho de CPU.

### W3 — paralelismo

Mesmo dataset com `max_parallel_workers_per_gather` variando em uma pequena matriz controlada.

### W4 — encerramento precoce

Seq Scan sob `LIMIT` para medir I/O especulativo e validar EXP-004.

## Coleta

Para cada execução:

- `EXPLAIN (ANALYZE, BUFFERS, SETTINGS, TIMING OFF, SUMMARY ON)`;
- snapshot de `pg_stat_io` antes/depois em laboratório silencioso;
- tempo wall-clock;
- CPU user/system;
- throughput e IOPS no SO;
- latência média/p95 do storage, quando disponível;
- métricas OBS-001 quando implementadas.

## Repetições

- warm-up separado e descartado;
- mínimo de 7 execuções por caso;
- usar mediana como número principal;
- registrar p95 e dispersão;
- alternar baseline/patch para reduzir efeito de drift térmico/cache/storage.

Exemplo de ordem: `A B A B A B ...`, não `AAAAAAA BBBBBBB`.

## Cache frio

Só executar limpeza de page cache do SO em uma VM/lab dedicado. Nunca incorporar `drop_caches` em script que possa rodar inadvertidamente em servidor compartilhado ou produção.

## Regra de decisão

Uma alteração só é melhoria quando o ganho é repetível e não depende de uma condição artificial não documentada. Resultados negativos também devem ser preservados para evitar repetir experimentos descartados.
