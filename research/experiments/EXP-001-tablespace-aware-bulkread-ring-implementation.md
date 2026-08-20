# PERF-001 — tablespace-aware bulk-read ring

Control: `research/obs-001-readstream`  
Experiment: `research/perf-001-tablespace-bulkread-ring`

The regular `GetAccessStrategy()` keeps upstream behavior. A new internal `GetAccessStrategyWithIOConcurrency()` accepts an already-resolved concurrency value. Heap bulk-read scans resolve `effective_io_concurrency` from the relation tablespace, except for catalog/early-startup cases where the global value is retained to mirror ReadStream's recursion-safety rule.

No catalog, wire-protocol or on-disk format changes are introduced.

A/B cases: global/tablespace 1/1, 1/8, 1/32, 16/1 and 16/32. Compare OBS-001 `max_ios`, `max_pins`, blocks/read, wait ratio, peak distance and elapsed time.
