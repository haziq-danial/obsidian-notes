---
tags: [performance, amdahls-law, parallelism]
---

# Amdahl's Law

> [!summary] Summary
> Amdahl's Law quantifies the maximum speedup achievable by improving (or parallelizing) only *part* of a system — the portion left unimproved caps the overall gain no matter how much you improve the rest.

## 1. The formula

$$ \text{Speedup} = \frac{1}{(1 - F) + \dfrac{F}{S}} $$

where:
- $F$ = fraction of the original execution time that *can* be improved/parallelized
- $S$ = speedup factor applied to that fraction
- $(1-F)$ = the unimproved (serial/sequential) fraction, unaffected by the improvement

As $S \to \infty$ (the improvable part becomes instantaneous):

$$ \text{Speedup}_{max} = \frac{1}{1-F} $$

> [!example] Diminishing returns from parallelizing part of a program
> A program spends 90% of its time in a parallelizable loop ($F = 0.9$) and 10% in inherently serial setup/teardown code.
> - With 4x speedup on the parallel part: Speedup = $1 / (0.1 + 0.9/4) = 1/0.325 ≈ 3.08x$
> - With infinite processors on the parallel part ($S \to \infty$): Speedup = $1/0.1 = 10x$ **maximum, no matter how many cores you add.**

This is the single most important reason "just add more cores" doesn't scale indefinitely — see [[Multicore and Multiprocessing]].

## 2. Applying Amdahl's Law to hardware design

The same formula applies to *any* partial improvement, not just parallelism:
- Speeding up the ALU only helps the fraction of time spent on ALU operations, not memory-bound stretches (see [[Memory Hierarchy]]).
- A faster cache only helps the fraction of accesses that would otherwise miss to a slower level (see [[Cache Memory]]'s AMAT formula, which is itself a form of Amdahl's-Law-style reasoning about where time is actually spent).

> [!tip] Design implication
> **Make the common case fast.** Amdahl's Law is the formal justification behind this classic architecture maxim (Hennessy & Patterson) — optimizing a rarely-executed path yields little overall benefit; optimizing whatever dominates execution time yields the most.

## 3. Gustafson's Law — the counterpoint

Amdahl's Law assumes a **fixed problem size** as you add processors, which understates parallel scalability for many real workloads. **Gustafson's Law** instead assumes the problem size *grows* with available compute (a common real-world pattern — more cores → run a bigger simulation, not just the same one faster):

$$ \text{Speedup} = (1 - F) + F \times N $$

where $N$ is the number of processors and $F$ is the parallel fraction *at that scale*. This gives a more optimistic (and often more realistic) picture for large-scale parallel systems like HPC clusters, where the interesting question is "how much bigger a problem can I solve in the same time?" rather than "how much faster can I solve this exact problem?"

## 4. Practical implications

- Before parallelizing, **profile** to find $F$ — parallelizing code that's only 20% of runtime can never yield more than a 1.25x overall speedup regardless of core count.
- Serial bottlenecks (I/O, locks, single-threaded initialization) become increasingly dominant as core count rises — this is why [[Multicore and Multiprocessing]]'s synchronization primitives (locks, atomics) and their contention cost matter so much at scale.
- The law applies recursively: after removing the biggest serial bottleneck, the *next* largest serial fraction becomes the new ceiling — optimization is iterative.

## See also
- [[Performance Metrics]]
- [[Multicore and Multiprocessing]]
- [[Pipelining]]

#performance #amdahls-law
