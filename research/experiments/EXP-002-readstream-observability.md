# EXP-002 — Observabilidade do ReadStream

Status: `planned`
Prioridade: P0
Risco esperado: baixo quando compilado apenas para laboratório

## Objetivo

Medir o comportamento adaptativo do ReadStream antes de alterar sua política.

## Contadores propostos

- `callback_blocks`: blocos pedidos ao callback;
- `cache_hits`: blocos que não exigiram wait/I/O;
- `io_blocks`: blocos cobertos por operações que exigiram I/O;
- `io_operations`: operações iniciadas;
- `combined_blocks`: soma dos blocos por operação;
- `wait_calls`: chamadas a `WaitReadBuffers()`;
- `distance_initial`, `distance_peak`;
- `distance_reductions`: reduções de look-ahead;
- `pin_limit_clamps`: vezes em que o limite de pins reduziu a distância;
- `forwarded_buffers`: buffers encaminhados após short read.

## Primeira implementação

Não alterar catálogo, ABI ou protocolo. Adicionar estrutura de counters somente no build experimental e emitir resumo em log DEBUG/LOG ou por um ponto de inspeção de desenvolvimento.

Depois do EXP-001, avaliar integração mais limpa com `Instrumentation`/`EXPLAIN` ou `pg_stat_io`.

## Por que vem antes das otimizações

Sem esses dados, uma melhora de tempo total não dirá se veio de:

- maior concorrência real;
- maior combinação de blocos;
- cache do SO;
- cache do PostgreSQL;
- menos waits;
- simples variação do storage.

A instrumentação deve ter custo mensurável próximo de zero quando desabilitada.
