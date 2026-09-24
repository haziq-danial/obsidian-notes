---
tags: [redis, performance, memory]
---

# Memory Optimization

> [!summary] Summary
> Since Redis holds its dataset in RAM, memory efficiency directly determines cost and capacity. This note collects the concrete techniques for shrinking a Redis dataset's memory footprint.

## 1. Prefer compact encodings by staying under thresholds

As covered per-type ([[Strings]], [[Lists]], [[Hashes]], [[Sets]], [[Sorted Sets]]), Redis automatically uses a compact, contiguous encoding (`listpack`/`intset`) for small collections and switches to a more memory-hungry-but-scalable encoding (`hashtable`/`skiplist`/`quicklist`) once configured thresholds are exceeded:

```
hash-max-listpack-entries 128
hash-max-listpack-value 64
set-max-intset-entries 512
set-max-listpack-entries 128
zset-max-listpack-entries 128
list-max-listpack-size 128
```

> [!tip] Bucketing strategy
> Splitting a huge logical collection into many smaller physical ones that each stay under these thresholds (e.g., hashing millions of user IDs into a few thousand buckets, each stored as one compact Hash) can cut memory usage by 5-10x compared to millions of individual top-level keys — because the compact encodings avoid the per-key and per-pointer overhead of the general-purpose hash table / skip list representations.

## 2. Minimize per-key overhead

Every top-level key carries overhead in Redis's global keyspace hash table (a `dictEntry`-like structure, pointers, etc.) independent of the value's own size. Implications:

- **Fewer, larger keys beat many tiny keys** where the data model allows it — this is the core justification for the Hash-bucketing pattern above.
- **Short key names save real memory at scale** — `u:1000` vs `user_profile_object:1000` adds up across millions of keys, though readability trade-offs should be weighed against the savings.

## 3. Use appropriate TTLs to bound growth

Setting TTLs (see [[Keys and Expiry]]) on cache-like or ephemeral data prevents unbounded keyspace growth from data that's logically temporary but never explicitly deleted by the application.

## 4. Choose the cheapest sufficient data type

- Use **Bitmaps** or **HyperLogLog** instead of Sets when the use case is a boolean flag per ID or an approximate distinct count (see [[Bitmaps and HyperLogLog]]) — both are dramatically cheaper than a Set for their respective specialized purposes.
- Use **integers, not numeric strings**, where possible — the `int` string encoding (see [[Strings]] §4) is cheaper than the general `raw`/`embstr` encodings.

## 5. `maxmemory` and eviction

Set an explicit `maxmemory` limit so Redis proactively manages memory rather than letting the OS start swapping (catastrophic for a latency-sensitive in-memory store) or the process getting OOM-killed. Pair it with an appropriate policy — see [[Eviction Policies]].

## 6. Inspecting memory usage

| Command | Purpose |
|---|---|
| `MEMORY USAGE key` | estimated bytes used by a specific key |
| `MEMORY STATS` | detailed internal memory breakdown |
| `MEMORY DOCTOR` | human-readable diagnostic suggestions |
| `INFO memory` | overall memory metrics (`used_memory`, fragmentation ratio, etc.) |
| `redis-cli --bigkeys` | sample the keyspace for unusually large keys/values |
| `redis-cli --memkeys` | sample the keyspace, sorted by per-key memory usage |

> [!warning] Memory fragmentation
> `used_memory_rss / used_memory` (the **fragmentation ratio** in `INFO memory`) above ~1.5 typically indicates the allocator (jemalloc by default) is holding onto more OS memory than Redis's own data actually needs — often from large keys being deleted/resized repeatedly. Redis's `activedefrag` setting can reclaim this incrementally in the background without a restart.

## 7. Sampling and profiling large keyspaces

For datasets too large to enumerate quickly, use `SCAN`-based sampling (see [[Keys and Expiry]] §2) combined with `MEMORY USAGE` on a representative sample to estimate where memory is actually going, rather than guessing from schema alone.

## See also
- [[Eviction Policies]]
- [[Strings]], [[Hashes]], [[Sets]], [[Sorted Sets]]
- [[Configuration]]

#redis #performance #memory
