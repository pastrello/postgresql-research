# PostgreSQL Research

Fork experimental baseado em PostgreSQL 18 para estudos controlados de segurança, desempenho, observabilidade e comportamento interno.

## Objetivo

Este repositório existe para testar hipóteses técnicas contra uma base PostgreSQL 18 conhecida, sempre comparando os resultados com um baseline não modificado.

O projeto não deve tratar uma alteração como melhoria apenas porque compila ou parece correta: toda mudança de desempenho deve ser acompanhada de benchmark reproduzível e toda mudança de segurança deve preservar os testes de regressão aplicáveis.

## Modelo de branches

- `main`: ponto estável do projeto e documentação aprovada.
- `upstream-18`: referência do PostgreSQL 18 original, sem alterações experimentais.
- `research/*`: experimentos isolados.
- `research/bootstrap`: preparação inicial da estrutura do laboratório.

## Áreas de pesquisa

- segurança e hardening;
- planner e estimativas de custo;
- executor;
- armazenamento, buffer manager e I/O;
- paralelismo;
- índices e grandes volumes;
- observabilidade e diagnóstico interno.

## Regra principal

Cada experimento deve registrar: hipótese, alteração, ambiente, workload, resultado, regressões observadas e decisão final (`accepted` ou `rejected`).

Consulte `docs/ROADMAP.md` e `docs/BENCHMARKING.md` antes de iniciar alterações no código-fonte.
