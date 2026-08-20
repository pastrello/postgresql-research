# SEC-001 — Limite configurável para mensagens grandes do frontend

Status: `research`
Prioridade: P1 hardening
Risco esperado: médio por compatibilidade

## Comportamento atual

`PQ_LARGE_MESSAGE_LIMIT` é definido como `MaxAllocSize - 1`. `SocketBackend()` usa esse limite para mensagens `Query`, `FunctionCall`, `Bind`, `Parse` e `CopyData`.

O protocolo valida o comprimento antes de alocar e encerra a conexão para comprimentos inválidos. Portanto, este item não é uma correção de validação ausente.

## Objetivo de hardening

Permitir que uma instalação fechada defina um teto significativamente menor para clientes autenticados, reduzindo a quantidade de memória que uma única mensagem defeituosa ou maliciosa pode pressionar em um backend.

## Design conservador

- default deve preservar integralmente o comportamento upstream;
- evitar um único limite se isso quebrar COPY/parametrização legítima;
- considerar limites separados para Query/Parse/Bind e CopyData;
- mudança via SIGHUP ou startup, não por usuário comum;
- erro claro no log contendo tipo da mensagem e limite, sem registrar conteúdo sensível.

## Testes

- simple query no limite e acima dele;
- extended query com parâmetro grande;
- COPY FROM STDIN;
- clientes libpq e drivers comuns;
- teste de recuperação de protocolo/conexão conforme semântica escolhida.

## Observação

Não classificar este item como vulnerabilidade do PostgreSQL. É um controle adicional para ambientes que preferem um teto operacional menor que o limite genérico de alocação.
