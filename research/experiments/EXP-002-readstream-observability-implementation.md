# OBS-001 — ReadStream instrumentation

Experimental branch: `research/obs-001-readstream`.

This implementation adds only backend-local counters to `ReadStream`; it changes no catalog, shared-memory layout, wire protocol or on-disk format. The counters are reported at `DEBUG1` when a stream is destroyed.

Counters: `reads_started`, `blocks_started`, `waited`, `no_wait`, `wait_calls`, and `peak_distance`. `no_wait` is intentionally not labelled as a cache hit because that conclusion is not guaranteed by the `StartReadBuffers()` contract.

To observe an isolated test session:

```sql
SET client_min_messages = DEBUG1;
```

Look for lines beginning with `PG18-R ReadStream stats:`. The useful derived metric for the first experiment is `blocks_started / reads_started`, alongside the wait ratio and peak look-ahead distance.

This instrumentation is research-only and should not be merged into a production-oriented branch as-is.
