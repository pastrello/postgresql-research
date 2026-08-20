# EXP-004 — Ramp-up do ReadStream em scans longos

Status: `planned-after-exp001`
Prioridade: P1
Risco esperado: médio

## Observação

`read_stream_begin_impl()` começa com `distance = 1`. O flag `READ_STREAM_FULL` pula essa fase e inicia assumindo até `io_combine_limit` buffers.

`heap_beginscan()` cria o stream do Seq Scan com `READ_STREAM_SEQUENTIAL | READ_STREAM_USE_BATCHING`, sem `READ_STREAM_FULL`.

## Por que isso não é simplesmente um bug

Um Seq Scan pode ser interrompido por um nó superior, como `LIMIT`, e iniciar uma janela grande de I/O poderia fazer trabalho desnecessário. A política conservadora tem boa justificativa.

## Hipótese experimental

Propagar do planner/executor uma indicação de alta probabilidade de consumir grande parte da relação e, apenas nesse caso, usar comportamento equivalente a `READ_STREAM_FULL`.

## Cenários

- full table scan sem LIMIT;
- aggregate que consome toda a relação;
- Seq Scan sob LIMIT pequeno;
- filtro altamente seletivo mas sem índice;
- serial e paralelo;
- cache frio e quente.

## Critério

A mudança precisa mostrar ganho mensurável nos scans longos sem aumentar significativamente I/O desperdiçado em consultas encerradas cedo.
