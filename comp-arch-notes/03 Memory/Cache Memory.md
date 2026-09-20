---
tags: [memory, cache]
---

# Cache Memory

> [!summary] Summary
> Caches are small, fast SRAM-based memories that transparently hold copies of recently/frequently used main-memory data, exploiting locality (see [[Memory Hierarchy]]) to hide most of DRAM's latency from the CPU.

## 1. Structure of a cache

A cache is organized into **lines (blocks)**, each holding:
- A **valid bit** (is this line's data meaningful?)
- A **tag** (which memory block is currently stored here?)
- The **data** itself (typically 32–128 bytes per line)
- (write-back caches only) a **dirty bit** (has this line been modified since it was loaded?)

An address is split into three fields to locate data in the cache:

```
| ------- Tag ------- | ---- Index ---- | -- Offset -- |
```
- **Offset**: selects a byte within the cache line.
- **Index**: selects which cache *set* to look in.
- **Tag**: compared against the stored tag to determine hit/miss.

## 2. Placement policies

![[cache-mapping.svg]]

| Policy | Where a block can go | Trade-off |
|---|---|---|
| Direct-mapped | Exactly one specific line: `(block #) mod (#lines)` | Fast, simple, but high conflict-miss rate |
| Fully associative | Any line at all | Best hit rate, but needs a comparator per line — expensive, slow for large caches |
| N-way set associative | Any of N lines within one specific set | The practical middle ground used by virtually all real caches (typically 4-, 8-, or 16-way) |

## 3. Replacement policies (for associative caches)

When a set is full and a new block must be brought in, which line gets evicted?
- **LRU (Least Recently Used)**: evict the line unused for the longest time — best hit rate typically, costlier to track exactly (often approximated in hardware).
- **FIFO**: evict the oldest-loaded line, regardless of use.
- **Random**: surprisingly competitive, trivial to implement, avoids adversarial worst cases.
- **Pseudo-LRU**: a cheap hardware approximation of true LRU used in most real associative caches.

## 4. Write policies

| Policy | Behavior | Trade-off |
|---|---|---|
| Write-through | Every write goes to cache *and* immediately to the next level down | Simpler, always consistent, but more write traffic |
| Write-back | Write only updates the cache line (marked dirty); written to lower memory only on eviction | Less traffic, faster, but needs dirty bits and more complex coherence |
| Write-allocate | On a write miss, load the block into cache first, then write | Pairs naturally with write-back |
| No-write-allocate | On a write miss, write directly to lower memory, bypassing the cache | Pairs naturally with write-through |

## 5. Types of misses — the "3 C's"

- **Compulsory (cold) miss**: the very first access to a block — unavoidable no matter the cache design.
- **Capacity miss**: the working set is larger than the cache, so blocks are evicted and later re-referenced even with optimal replacement.
- **Conflict miss**: multiple blocks map to the same set (in non-fully-associative caches) and evict each other despite spare capacity elsewhere in the cache.

(A fourth, "coherence miss," is sometimes added for multi-core systems — see [[Multicore and Multiprocessing]] for cache-coherence protocols like MESI.)

## 6. Multi-level caches and split caches

Modern CPUs use **split L1** instruction/data caches (feeding the pipeline every cycle without structural hazards, see [[Pipeline Hazards]]) backed by a larger **unified L2**, often backed further by a large **shared L3** across cores (see [[Memory Hierarchy]]).

## 7. Cache-friendly code

> [!tip] Why loop order matters
> Iterating a 2D array in **row-major order** (matching how it's laid out in memory for C/C++) exploits spatial locality — each cache line loaded serves several consecutive accesses. Iterating column-major instead can multiply cache misses by the array's row length, causing dramatic (often 10x+) slowdowns despite doing the *exact same number of arithmetic operations*.

```c
// Cache-friendly: sequential access matches memory layout
for (i = 0; i < N; i++)
  for (j = 0; j < N; j++)
    sum += A[i][j];   // A[i][j] and A[i][j+1] are adjacent in memory

// Cache-unfriendly: strides through memory, evicting lines before reuse
for (j = 0; j < N; j++)
  for (i = 0; i < N; i++)
    sum += A[i][j];   // A[i][j] and A[i+1][j] are N elements apart
```

## See also
- [[Memory Hierarchy]]
- [[Virtual Memory]]
- [[Multicore and Multiprocessing]] — cache coherence across cores

#memory #cache
