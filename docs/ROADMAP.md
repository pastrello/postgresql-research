# Roadmap inicial

## Fase 0 — Baseline

- importar e identificar exatamente a versão PostgreSQL 18 utilizada;
- registrar compilador, flags, kernel, filesystem, CPU, RAM e storage;
- executar regression tests;
- criar conjunto inicial de benchmarks reproduzíveis.

## Fase 1 — Auditoria operacional

- revisar configuração de build;
- mapear warnings relevantes de compilação;
- identificar caminhos críticos de CPU e I/O;
- levantar pontos de instrumentação úteis.

## Fase 2 — Segurança

- compilações instrumentadas com sanitizers quando suportadas;
- revisão de limites, buffers e conversões numéricas em áreas escolhidas;
- fuzzing seletivo de parsers/interfaces adequadas;
- revisão de superfícies de autenticação, protocolo e extensões conforme escopo.

## Fase 3 — Observabilidade

- melhorar explicabilidade de decisões do planner;
- instrumentar métricas experimentais sem alterar o comportamento padrão;
- medir overhead da instrumentação.

## Fase 4 — Grandes volumes

- scans sequenciais e bitmap;
- prefetch e comportamento de I/O;
- buffer manager;
- agregações e sorts;
- índices B-tree e BRIN;
- paralelismo.

## Fase 5 — Experimentos avançados

Somente após um baseline confiável: heurísticas adaptativas, mudanças de planner, estratégias alternativas de prefetch/cache e outras alterações de maior risco.
