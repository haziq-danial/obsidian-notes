---
tags: [parallelism, flynns-taxonomy]
---

# Flynn's Taxonomy

> [!summary] Summary
> Michael Flynn's 1966 classification sorts computer architectures by how many independent streams of **instructions** and **data** they process simultaneously — still the standard vocabulary for describing parallel hardware.

![[flynn-taxonomy.svg]]

## The four categories

| | Single Data | Multiple Data |
|---|---|---|
| **Single Instruction** | **SISD** — classic single-core scalar CPU | **SIMD** — one operation applied to many data elements at once |
| **Multiple Instruction** | **MISD** — rare, mainly fault-tolerant redundant computation | **MIMD** — independent processors each with their own instruction stream |

### SISD (Single Instruction, Single Data)
The traditional von Neumann model (see [[Von Neumann vs Harvard Architecture]]): one instruction stream operating on one data stream at a time. A single-core, non-vectorized CPU pipeline (see [[Pipelining]]) is SISD, even though pipelining overlaps instruction *stages* — it's still one logical instruction stream.

### SIMD (Single Instruction, Multiple Data)
One instruction applies the same operation to multiple data elements in parallel — ideal for data-parallel workloads (image/audio processing, linear algebra, graphics).

- **Vector/SIMD extensions**: SSE, AVX/AVX-512 (x86), NEON/SVE (ARM) — pack multiple values into wide registers (e.g., 512 bits = 16×32-bit floats) and apply one instruction to all of them.
- **GPUs** are built almost entirely around massive SIMD/SIMT execution — see [[GPU Architecture]].

> [!example] SIMD speedup
> Adding two arrays of 8 floats: a scalar loop issues 8 separate ADD instructions. An AVX instruction operating on 256-bit registers (8×32-bit floats) does it in **one** instruction — up to 8x throughput for this operation, given properly aligned/vectorizable code.

### MISD (Multiple Instruction, Single Data)
Multiple processing units apply *different* operations to the *same* data stream — genuinely rare in general-purpose computing. Mainly seen in fault-tolerant systems (e.g., spacecraft flight computers) where redundant units perform different computations on identical input and vote on the result to detect hardware faults.

### MIMD (Multiple Instruction, Multiple Data)
Independent processors, each executing its own instruction stream on its own data — the model for essentially all modern multicore CPUs, multiprocessor systems, and clusters. See [[Multicore and Multiprocessing]].

- **Shared-memory MIMD**: all processors access a common address space (typical multicore CPU).
- **Distributed-memory MIMD**: each processor has private memory, communicating via message passing (compute clusters, MPI-based supercomputers).

## Why the taxonomy still matters

A modern high-end system is layered: a **MIMD** collection of cores, each core capable of **SIMD** execution on vector units, executing what is logically **SISD** instruction-stream control flow per thread. Understanding which layer a performance optimization targets (vectorization vs. multithreading vs. distributed scaling) starts with placing it correctly in this taxonomy.

## See also
- [[Multicore and Multiprocessing]]
- [[GPU Architecture]]
- [[Von Neumann vs Harvard Architecture]]

#parallelism #flynns-taxonomy
