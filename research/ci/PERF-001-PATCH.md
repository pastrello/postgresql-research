# PERF-001 patch result

- patch rc: 0
- result: PASS

```text
src/include/storage/bufmgr.h: prototype: matches=1
src/backend/storage/buffer/freelist.c: wrapper/helper: matches=1
src/backend/storage/buffer/freelist.c: ring calculation: matches=1
src/backend/access/heap/heapam.c: catalog include: matches=1
src/backend/access/heap/heapam.c: miscadmin include: matches=1
src/backend/access/heap/heapam.c: heap strategy: matches=1
```
diff-check rc=0
