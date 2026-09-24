---
tags: [redis, data-structures, bitmaps, hyperloglog]
---

# Bitmaps and HyperLogLog

> [!summary] Summary
> Two specialized, extremely memory-efficient tools for a common class of problem: tracking huge numbers of boolean flags (bitmaps) or estimating huge distinct counts (HyperLogLog) using vastly less memory than a naive Set-based approach.

## 1. Bitmaps

A bitmap isn't a separate data type — it's just bit-level operations on a regular [[Strings|String]], treating its bytes as a raw bit array.

| Command | Effect |
|---|---|
| `SETBIT key offset 0\|1` | set a single bit |
| `GETBIT key offset` | read a single bit |
| `BITCOUNT key [start end]` | count set bits (optionally within a byte or bit range) |
| `BITOP AND\|OR\|XOR\|NOT dest key [key ...]` | bitwise operations across bitmaps, stored into `dest` |
| `BITPOS key bit` | find the first bit set to 0 or 1 |
| `BITFIELD key ...` | pack multiple small integer counters into one bitmap-backed string |

> [!example] Daily active users with bitmaps
> ```
> SETBIT active:2024-01-01 1000 1     # user 1000 was active
> SETBIT active:2024-01-02 1000 1
> BITCOUNT active:2024-01-01           # count of active users that day
> BITOP AND active:both active:2024-01-01 active:2024-01-02   # active on BOTH days
> BITCOUNT active:both
> ```
> Tracking a boolean flag per user ID this way costs **1 bit per user** — a million users is ~125KB, versus megabytes for an equivalent [[Sets|Set]] of user-ID strings.

## 2. HyperLogLog (HLL)

A probabilistic algorithm for estimating the number of **distinct** elements added to a set, using a small, **fixed** amount of memory (~12KB) regardless of whether you've counted a thousand or a billion distinct items.

| Command | Effect |
|---|---|
| `PFADD key element [element ...]` | add element(s) to the estimator |
| `PFCOUNT key [key ...]` | estimated cardinality (of one key, or the union of several) |
| `PFMERGE dest key [key ...]` | merge several HLLs into one |

> [!warning] The trade-off: approximation
> HyperLogLog trades exactness for memory: it has a **standard error of ~0.81%**. It also only supports counting — there's no way to retrieve the actual elements back out (unlike a `SCARD` on a real Set). Use it specifically when you need "roughly how many unique X" at massive scale (unique visitors, unique search queries) and don't need the member list itself.

> [!example] Unique visitor counting at scale
> ```
> PFADD visitors:2024-01-01 user123 user456 user789
> PFCOUNT visitors:2024-01-01              → ~3
> ```
> For 100 million unique visitors, a Set would need tens of gigabytes; a HyperLogLog needs ~12KB — a compression ratio only possible by giving up exactness and the ability to enumerate members.

### How it works (conceptually)
HLL is based on the observation that, in a stream of random hash values, the **longest run of leading zero bits** observed gives a statistical estimate of how many distinct values have been hashed (rarer long runs imply more distinct inputs). Splitting hashes into many buckets ("registers") and averaging the estimate across them (with bias correction) dramatically reduces variance — that's the algorithmic core behind the ~12KB, fixed-size implementation.

## 3. When to reach for these vs a Set

| Need | Best fit |
|---|---|
| Exact membership + ability to list all members | [[Sets]] |
| Exact count, small-to-medium cardinality | `SCARD` on a Set |
| Approximate distinct count, huge cardinality, memory-constrained | HyperLogLog |
| Per-entity boolean flags at massive scale (millions of IDs) | Bitmaps |

## See also
- [[Strings]]
- [[Sets]]
- [[Memory Optimization]]

#redis #data-structures #bitmaps #hyperloglog
