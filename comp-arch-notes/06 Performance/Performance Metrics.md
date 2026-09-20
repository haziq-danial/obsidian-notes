---
tags: [performance, metrics]
---

# Performance Metrics

> [!summary] Summary
> "Faster" needs a precise definition before it can be measured or optimized. This note covers the core equations relating clock rate, CPI, instruction count, and execution time, plus the standard benchmarking pitfalls.

## 1. The fundamental CPU performance equation

$$ \text{CPU Time} = \text{Instruction Count} \times \text{CPI} \times \text{Clock Cycle Time} $$

or equivalently, since Clock Cycle Time = 1 / Clock Rate:

$$ \text{CPU Time} = \frac{\text{Instruction Count} \times \text{CPI}}{\text{Clock Rate}} $$

Every major architectural decision in this vault trades off one of these three factors against another:

| Factor | Improved by | Sometimes worsened by |
|---|---|---|
| Instruction Count | Better compiler optimization, richer ISA (CISC) | — |
| CPI (Cycles Per Instruction) | [[Pipelining]], [[Superscalar and Out-of-Order Execution]] | Hazards/stalls (see [[Pipeline Hazards]]), cache misses |
| Clock Rate | Shorter pipeline stages, better process technology | Deeper pipelines increase misprediction penalty |

> [!example] Comparing two designs
> Design A: 1 GHz clock, CPI = 1.2, 10⁹ instructions → time = 10⁹ × 1.2 / 10⁹ = **1.2 s**
> Design B: 1.5 GHz clock, CPI = 1.8, 10⁹ instructions → time = 10⁹ × 1.8 / (1.5×10⁹) = **1.2 s**
> Identical clock rate isn't the whole story — CPI matters just as much, which is why "clock speed" alone is a poor performance proxy (a lesson the industry learned the hard way with the Pentium 4's deep pipeline chasing GHz at the cost of CPI).

## 2. MIPS and FLOPS — and why they're misleading alone

- **MIPS** (Million Instructions Per Second) = Clock Rate / (CPI × 10⁶). Misleading across different ISAs — a RISC CPU naturally executes *more* simple instructions to do the same work as fewer complex CISC instructions (see [[Instruction Set Architecture]]), so higher MIPS doesn't mean faster in wall-clock terms.
- **FLOPS** (Floating-point Operations Per Second) is more meaningful for numerical/scientific workloads, but still workload-dependent (a chip's peak FLOPS is rarely sustained on real code due to memory bandwidth limits — see [[Memory Hierarchy]]).

> [!warning] The only trustworthy metric
> **Execution time on a representative workload** is the only performance metric that can't be gamed by architectural quirks. Everything else (clock rate, MIPS, FLOPS, CPI alone) is a proxy that can mislead when comparing different architectures.

## 3. Benchmarks

- **Microbenchmarks**: isolate one specific operation (e.g., memory bandwidth, branch misprediction cost) — useful for architectural analysis, not representative of whole applications.
- **Benchmark suites** (e.g., SPEC CPU): a standardized collection of real-ish programs, aggregated (often via geometric mean, which is robust to the choice of reference machine) into a single comparable score.

> [!warning] Benchmarking pitfalls
> - **Benchmarking the compiler, not the hardware**: aggressive, benchmark-specific compiler flags can misrepresent real-world performance.
> - **Amdahl's Law effects hiding in aggregate scores**: a huge speedup on one sub-benchmark can be diluted by neglecting the rest of the suite — see [[Amdahls Law]].
> - **Non-representative workloads**: a benchmark suite from a decade ago may not reflect today's actual software mix (e.g., ML workloads barely resembled in classic SPEC suites).

## 4. Latency vs throughput vs bandwidth

| Term | Definition | Example |
|---|---|---|
| Latency | Time for a single operation to complete | Time for one memory load to return data |
| Throughput | Operations completed per unit time | Instructions retired per second |
| Bandwidth | Data volume transferred per unit time | GB/s from DRAM |

Pipelining (see [[Pipelining]]) is the canonical example of improving throughput without improving (and sometimes slightly worsening) latency.

## 5. Power and energy

$$ \text{Dynamic Power} \approx C \times V^2 \times f $$

where $C$ = switched capacitance, $V$ = supply voltage, $f$ = clock frequency. This is exactly the equation behind the shift to multicore covered in [[Multicore and Multiprocessing]] §1 — once voltage couldn't keep dropping fast enough to offset rising frequency (end of Dennard scaling), raw frequency scaling stopped being power-efficient.

**Energy** (not just power) matters for battery-powered/mobile systems and datacenter costs:

$$ \text{Energy} = \text{Power} \times \text{Time} $$

A design that finishes a fixed amount of work faster, even at higher instantaneous power, can still use *less total energy* — the "race to idle" principle behind many mobile SoC power-management strategies.

## See also
- [[Amdahls Law]]
- [[Pipelining]]
- [[Memory Hierarchy]]
- [[Multicore and Multiprocessing]]

#performance #metrics
