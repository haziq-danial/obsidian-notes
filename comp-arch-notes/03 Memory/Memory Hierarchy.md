---
tags: [memory, memory-hierarchy]
---

# Memory Hierarchy

> [!summary] Summary
> No single memory technology is simultaneously fast, large, and cheap. The memory hierarchy exploits this by layering small/fast memory close to the CPU and large/slow/cheap memory further away, relying on **locality of reference** to make the common case fast.

![[memory-hierarchy-pyramid.svg]]

## 1. Why a hierarchy works: locality

- **Temporal locality**: if a location was accessed recently, it's likely to be accessed again soon (e.g., loop variables, function-local data).
- **Spatial locality**: if a location was accessed, nearby locations are likely to be accessed soon too (e.g., sequential array traversal, instruction fetch of sequential code).

Caches and prefetchers exploit both: fetching a whole **cache line/block** (spatial) and keeping recently-used data around (temporal).

## 2. The levels

| Level | Typical size | Typical latency | Managed by |
|---|---|---|---|
| Registers | ~32 × 64-bit | 0 cycles (part of pipeline) | Compiler |
| L1 Cache | 32–64 KB (I+D) | ~4 cycles | Hardware |
| L2 Cache | 256 KB – 1 MB | ~12 cycles | Hardware |
| L3 Cache | several MB – tens of MB, shared | ~40 cycles | Hardware |
| Main memory (DRAM) | GBs | ~100–300 cycles | Hardware + OS (paging) |
| SSD | hundreds of GB – TBs | ~10,000s of cycles (µs) | OS (filesystem) |
| HDD | TBs | ~millions of cycles (ms) | OS (filesystem) |

Note the massive latency gap between each level — this is why keeping data as high in the hierarchy as possible dominates real-world performance, often far more than raw clock speed.

## 3. Average Memory Access Time (AMAT)

$$ \text{AMAT} = \text{Hit time} + \text{Miss rate} \times \text{Miss penalty} $$

For a multi-level hierarchy, this recurses:

$$ \text{AMAT} = T_1 + MR_1 \times (T_2 + MR_2 \times (T_3 + MR_3 \times T_{mem})) $$

> [!example] AMAT calculation
> L1 hit time = 1 cycle, L1 miss rate = 5%. L2 hit time = 10 cycles, L2 miss rate (of L1 misses) = 20%. Memory access = 100 cycles.
> $$ \text{AMAT} = 1 + 0.05 \times (10 + 0.20 \times 100) = 1 + 0.05 \times 30 = 2.5 \text{ cycles} $$

This equation is the single most important formula for reasoning about [[Cache Memory]] design decisions.

## 4. Inclusive vs exclusive vs NINE cache hierarchies

- **Inclusive**: L2 contains a superset of everything in L1 (simplifies cache-coherence checks across cores at the cost of wasted capacity).
- **Exclusive**: a line lives in exactly one level at a time (maximizes effective capacity, more complex to manage on eviction).
- **NINE (Non-Inclusive Non-Exclusive)**: no guarantee either way — common in practice as a simpler compromise.

## 5. Where the hierarchy continues below RAM

- [[Virtual Memory]] extends the hierarchy conceptually to disk/SSD via demand paging — main memory itself acts as a "cache" for the (much larger) virtual address space backed by storage.
- [[Memory Technologies]] covers the physical technologies (SRAM, DRAM, Flash) that implement each level.

## See also
- [[Cache Memory]]
- [[Virtual Memory]]
- [[Memory Technologies]]
- [[Performance Metrics]]

#memory #memory-hierarchy
