# PostgreSQL 18.6 Research Audit

Esta árvore foi adicionada na branch `research/audit-18.6` sem modificar o comportamento do PostgreSQL.

## Documentos

- `AUDIT-18.6.md`: auditoria inicial e priorização.
- `BENCHMARK-BASELINE.md`: regras para comparação A/B.
- `experiments/EXP-001-tablespace-aware-bulkread-ring.md`: primeiro experimento funcional.
- `experiments/EXP-002-readstream-observability.md`: instrumentação necessária.
- `experiments/EXP-003-aio-aware-seqscan-costing.md`: costing do planner, bloqueado até obter medições.
- `experiments/EXP-004-readstream-full-scan-ramp.md`: estudo do ramp-up de scans longos.
- `security/SEC-001-frontend-message-limit.md`: hardening de tamanho de mensagens.
- `security/SEC-002-dynamic-library-allowlist.md`: hardening opcional do loader.

## Estado

Nenhuma alteração funcional foi aplicada nesta branch nesta etapa. O primeiro código deverá ser a instrumentação de EXP-002 ou o patch pequeno de EXP-001, sempre em branch filha dedicada.
