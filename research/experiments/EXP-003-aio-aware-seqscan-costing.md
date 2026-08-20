# EXP-003 — Costing de Seq Scan consciente de AIO

Status: `blocked-by-measurement`
Prioridade: P1
Risco esperado: alto (muda escolhas do planner)

## Observação atual

`cost_seqscan()` usa:

`disk_run_cost = spc_seq_page_cost * baserel->pages`

Para scans paralelos o custo de CPU é dividido pelo divisor paralelo, mas o custo de disco não é amortizado. O comentário explica a suposição de que o readahead do sistema operacional já reduz boa parte do ganho potencial.

No PostgreSQL 18, entretanto, há `ReadStream`, combinação de blocos e AIO configurável. O runtime conhece `effective_io_concurrency` por tablespace, enquanto `cost_seqscan()` não usa essa informação.

## O que NÃO fazer

Não dividir simplesmente `disk_run_cost` por `effective_io_concurrency`. Isso superestimaria o benefício, ignoraria latência, profundidade real da fila, cache, tamanho das operações, throughput do dispositivo e contenção entre workers.

## Experimento

Depois de OBS-001 e PERF-001:

1. coletar curvas de throughput versus concorrência;
2. comparar `sync`, `worker` e `io_uring` quando suportados;
3. medir storage local e/ou remoto disponível no laboratório;
4. estimar uma função conservadora de ganho marginal;
5. testar se ela melhora escolhas entre Seq Scan, Bitmap Heap Scan e Index Scan sem criar regressões.

## Critério de aceitação

Só criar patch de planner se houver um padrão repetível que o modelo atual não consiga representar adequadamente usando os GUCs existentes (`seq_page_cost`, `random_page_cost`, `effective_cache_size`, custos de CPU e paralelismo).
