---
tags: [redis, data-structures, sorted-sets, zset]
---

# Sorted Sets

> [!summary] Summary
> A Sorted Set (ZSet) is like a [[Sets|Set]] where every member also has a floating-point **score**, and the set is always kept ordered by that score. This combination — unique members, O(log n) ordered operations, O(1) score lookup — makes it Redis's most powerful data type for ranking, scheduling, and range-query use cases.

## 1. Basic operations

| Command | Effect |
|---|---|
| `ZADD key score member [score member ...]` | add or update member(s) with a score |
| `ZSCORE key member` | get a member's score |
| `ZINCRBY key increment member` | atomically adjust a member's score |
| `ZREM key member [member ...]` | remove member(s) |
| `ZCARD key` | number of members |
| `ZRANK key member` / `ZREVRANK key member` | member's position (ascending / descending) |

## 2. Range queries — the real power of ZSets

| Command | Effect |
|---|---|
| `ZRANGE key start stop [WITHSCORES]` | members by rank (index) range |
| `ZREVRANGE key start stop` | same, descending order |
| `ZRANGEBYSCORE key min max` | members whose score falls in `[min, max]` |
| `ZRANGEBYLEX key min max` | members in lexicographic range (when all scores are equal — a trick for building sorted string indexes) |
| `ZCOUNT key min max` | count of members within a score range |
| `ZREMRANGEBYRANK` / `ZREMRANGEBYSCORE` | bulk-remove members by rank or score range |
| `ZPOPMIN` / `ZPOPMAX` | remove and return the lowest/highest-scoring member(s) |
| `BZPOPMIN` / `BZPOPMAX` | blocking versions — useful for priority-queue workers |

## 3. Internal structure: skip list + hash table

![[redis-data-types.svg]]

A Sorted Set combines **two structures internally** to get the best of both:
- A **hash table** mapping member → score, giving O(1) `ZSCORE` lookups.
- A **skip list** (a probabilistic, multi-level linked structure) keeping members ordered by score, giving O(log n) range and rank operations.

```mermaid
graph LR
    subgraph Skip List (ordered by score)
        L4["Level 4: → carol(70) ------------→ NIL"]
        L2["Level 2: → bob(85) → carol(70) → NIL"]
        L1["Level 1: → alice(99) → bob(85) → carol(70) → NIL"]
    end
```

This dual structure is exactly why ZSets support both "give me the score for X" (O(1), via the hash table) and "give me everyone ranked between position 10 and 20" (O(log n) + result size, via the skip list) — a combination no single simpler structure achieves as efficiently.

For small sorted sets, Redis instead uses the same compact `listpack` encoding as small Hashes/Lists, only switching to the full skiplist+hashtable representation once size thresholds are exceeded (a [[Memory Optimization]] trade-off, same principle as other types).

## 4. Common use cases

- Real-time leaderboards (score = points, member = player — see [[Leaderboards]])
- Priority queues (score = priority or scheduled-run timestamp; workers `ZPOPMIN`)
- Rate limiting with sliding windows (score = request timestamp, `ZREMRANGEBYSCORE` to expire old entries — see [[Rate Limiting]])
- Time-series-like data (score = timestamp, member = event ID)
- Geospatial indexing (see [[Geospatial Indexes]] — implemented as a Sorted Set scored by geohash under the hood)
- Autocomplete / secondary indexes via `ZRANGEBYLEX`

## See also
- [[Leaderboards]]
- [[Rate Limiting]]
- [[Geospatial Indexes]]
- [[Sets]]

#redis #data-structures #sorted-sets
