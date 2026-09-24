---
tags: [redis, data-structures, sets]
---

# Sets

> [!summary] Summary
> A Redis Set is an unordered collection of unique strings, supporting O(1) membership tests and full set algebra (union, intersection, difference) computed server-side.

## 1. Basic operations

| Command | Effect |
|---|---|
| `SADD key m1 m2 ...` | add member(s) |
| `SREM key m1 m2 ...` | remove member(s) |
| `SISMEMBER key member` | O(1) membership test |
| `SMISMEMBER key m1 m2 ...` | membership test for multiple members at once |
| `SMEMBERS key` | all members (avoid on huge sets — use `SSCAN`) |
| `SCARD key` | cardinality (member count) |
| `SPOP key [count]` | remove and return random member(s) |
| `SRANDMEMBER key [count]` | return random member(s) without removing |

## 2. Set algebra

| Command | Effect |
|---|---|
| `SINTER k1 k2 ...` | members present in **all** sets |
| `SUNION k1 k2 ...` | members present in **any** set |
| `SDIFF k1 k2 ...` | members in the first set but not the others |
| `SINTERSTORE dest k1 k2 ...` | intersect and store the result under `dest` |
| `SUNIONSTORE`, `SDIFFSTORE` | same idea for union/difference |
| `SINTERCARD numkeys k1 k2 ... [LIMIT n]` | count of the intersection without materializing it |

> [!example] Tag-based filtering
> Find posts tagged both "redis" and "database":
> ```
> SADD tag:redis post:1 post:2 post:3
> SADD tag:database post:2 post:3 post:4
> SINTER tag:redis tag:database   → post:2, post:3
> ```
> This kind of ad-hoc, multi-way intersection would require either an application-side loop or a relational join in a traditional database — Redis computes it directly, server-side, in one round trip.

## 3. Internal encoding: intset vs listpack vs hashtable

| Encoding | Used when |
|---|---|
| `intset` | every member is an integer, and the set is small — a sorted array of ints, extremely compact |
| `listpack` | small sets with non-integer members (Redis 7.2+) |
| `hashtable` | once the set grows beyond the configured thresholds |

Sets of pure integers (e.g., numeric IDs) stay in the highly compact `intset` encoding far longer than mixed-type sets — a [[Memory Optimization]] consideration when choosing whether to store IDs as ints vs strings.

## 4. Common use cases

- Tagging systems (as above)
- Unique visitor tracking (`SADD visitors:2024-01-01 user123`, then `SCARD` for a unique count — for very large scales, see [[Bitmaps and HyperLogLog]] instead)
- Relationship graphs — followers/following sets, mutual-friends via `SINTER`
- Deduplication (`SADD` returns whether a member was newly added, an O(1) "have I seen this before?" check)

## See also
- [[Sorted Sets]]
- [[Bitmaps and HyperLogLog]]
- [[Memory Optimization]]

#redis #data-structures #sets
