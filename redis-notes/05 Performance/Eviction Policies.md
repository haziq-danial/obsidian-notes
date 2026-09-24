---
tags: [redis, performance, eviction, maxmemory]
---

# Eviction Policies

> [!summary] Summary
> When Redis hits its configured `maxmemory` limit, an eviction policy determines what happens next: refuse new writes, or automatically remove existing keys to make room. Choosing the right policy depends on whether Redis is being used as a strict cache or a primary datastore.

![[eviction-policies.svg]]

## 1. Setting a memory limit

```
maxmemory 4gb
maxmemory-policy allkeys-lru
```

Without `maxmemory` set, Redis will keep allocating memory until the OS itself runs out — risking a slow, unpredictable death by swapping or an OOM-killer termination, rather than a controlled, immediate response. Setting an explicit limit is standard practice in any production deployment.

## 2. The eviction policies

| Policy | Eligible keys | Eviction criterion |
|---|---|---|
| `noeviction` (default) | none | writes fail with an error once full; reads still work |
| `allkeys-lru` | all keys | evict least-recently-used |
| `volatile-lru` | keys with a TTL only | evict least-recently-used among those with a TTL |
| `allkeys-lfu` | all keys | evict least-frequently-used |
| `volatile-lfu` | keys with a TTL only | evict least-frequently-used among those with a TTL |
| `allkeys-random` | all keys | evict at random |
| `volatile-random` | keys with a TTL only | evict at random among those with a TTL |
| `volatile-ttl` | keys with a TTL only | evict the key with the nearest expiry time first |

## 3. Choosing a policy

> [!example] Pure cache use case
> An application uses Redis purely to cache database query results, all set with a TTL as a safety net, expecting Redis to hold whatever fits and drop the rest transparently. `allkeys-lru` (or `allkeys-lfu` if access patterns are bursty/repetitive rather than recency-based) is the natural fit — any key can be evicted, prioritizing whatever's actually being used.

> [!example] Primary datastore use case
> An application stores data in Redis that has no other copy (e.g., real-time counters that aren't persisted elsewhere). Silently losing this data to eviction would be a correctness bug, not just a cache miss. `noeviction` is appropriate — better to have writes fail loudly (and be handled/alerted on) than silently lose data.

> [!example] Mixed workload
> Some keys are critical (no TTL, must never be evicted) while others are disposable cache entries (TTL set). `volatile-lru` or `volatile-ttl` lets Redis evict only the disposable, TTL-bearing keys, leaving permanent keys untouched even under memory pressure — though this requires disciplined key design (consistently setting/not-setting TTLs according to that intent).

## 4. LRU vs LFU

- **LRU (Least Recently Used)** approximates true LRU using a **sampling algorithm**: rather than tracking exact access order for every key (expensive), Redis samples a small random set of keys (`maxmemory-samples`, default 5) and evicts the most stale among just that sample — a good-enough approximation at a fraction of the bookkeeping cost. Higher `maxmemory-samples` trades a small CPU/memory cost for a closer approximation to true LRU.
- **LFU (Least Frequently Used)** tracks an approximate access-frequency counter per key (using a probabilistic, logarithmic counter to keep the overhead tiny) and decays it over time, better suited to workloads where "used often over time" is a better retention signal than "used most recently" — e.g., a small set of very popular items that are momentarily not accessed shouldn't be evicted just because of a brief lull.

## 5. What happens during eviction

Eviction is checked **before executing** each write command once `maxmemory` is exceeded: Redis evicts keys according to policy until memory is back under the limit, then proceeds with the command. This happens synchronously on the main thread — an unusually aggressive eviction storm (many keys needing removal at once) can itself introduce latency, another reason to size `maxmemory` with realistic headroom rather than running right at the edge.

## See also
- [[Memory Optimization]]
- [[Keys and Expiry]]
- [[Caching Patterns]]

#redis #performance #eviction
