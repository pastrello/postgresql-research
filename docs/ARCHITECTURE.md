# Arquitetura de trabalho

Este documento descreve como o fork experimental será organizado ao redor da árvore oficial do PostgreSQL 18.

## Princípios

1. Manter uma referência upstream identificável e reproduzível.
2. Isolar experimentos em branches próprias.
3. Evitar misturar otimizações independentes no mesmo teste.
4. Preservar compatibilidade com os testes de regressão do PostgreSQL sempre que possível.
5. Medir antes e depois de cada alteração.

## Subsistemas prioritários

- `src/backend/optimizer/` — planner, custos e escolha de planos.
- `src/backend/executor/` — execução dos planos.
- `src/backend/access/` — métodos de acesso e índices.
- `src/backend/storage/` — buffer manager, locks e I/O.
- `src/backend/utils/` — infraestrutura compartilhada.
- `src/include/` — estruturas e contratos internos.

## Fluxo de um experimento

1. Registrar hipótese em `research/ideas/`.
2. Criar branch `research/<tema>` a partir da referência apropriada.
3. Executar baseline.
4. Implementar a menor alteração possível.
5. Rodar testes de regressão.
6. Rodar benchmark reproduzível.
7. Coletar perfil de CPU, I/O, memória e locks quando necessário.
8. Comparar contra o baseline.
9. Mover a conclusão para `research/accepted/` ou `research/rejected/`.

## Política de alterações

Mudanças em segurança e correção têm prioridade sobre micro-otimizações. Otimizações que aumentem complexidade devem demonstrar benefício mensurável e consistente em workloads definidos.
