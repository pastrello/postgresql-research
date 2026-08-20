# Metodologia de benchmark

Nenhuma alteração de desempenho será classificada como melhoria sem comparação reproduzível com o PostgreSQL 18 de referência.

## Para cada teste registrar

- commit/branch;
- versão exata do PostgreSQL;
- configuração do build;
- `postgresql.conf` relevante;
- hardware/VM;
- kernel e filesystem;
- tamanho do dataset;
- estado do cache (quando relevante);
- SQL/workload;
- número de repetições;
- latência, throughput e dispersão;
- CPU, memória, I/O e waits relevantes.

## Execução

Sempre que possível:

1. aquecimento controlado;
2. várias repetições do baseline;
3. várias repetições do candidato;
4. alternância da ordem para reduzir viés temporal;
5. comparação por mediana e percentis, não apenas uma execução isolada.

## Ferramentas candidatas

- `pgbench`;
- `EXPLAIN (ANALYZE, BUFFERS, WAL, SETTINGS)`;
- `perf`;
- `iostat`;
- `vmstat`;
- métricas internas do PostgreSQL.

Scripts oficiais do projeto ficam em `benchmarks/scripts/` e consultas em `benchmarks/sql/`.
