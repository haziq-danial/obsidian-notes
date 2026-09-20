---
tags: [glossary, reference]
---

# Glossary

> [!summary] Summary
> Quick-reference definitions for terms used throughout the vault. Each term links to the note where it's covered in depth.

### A
- **AMAT (Average Memory Access Time)** — expected time per memory access accounting for hit time, miss rate, and miss penalty across cache levels. See [[Memory Hierarchy]].
- **Amdahl's Law** — formula capping overall speedup by the fraction of a task that remains unimproved. See [[Amdahls Law]].
- **ALU (Arithmetic Logic Unit)** — combinational circuit performing arithmetic/logic operations. See [[Combinational Logic Circuits]].
- **Associativity** — how many cache lines a memory block may occupy (direct-mapped, N-way, fully associative). See [[Cache Memory]].

### B
- **Branch prediction** — hardware guessing a branch's outcome before it's resolved, to avoid pipeline stalls. See [[Pipeline Hazards]].
- **Branch Target Buffer (BTB)** — cache of recent branch target addresses. See [[Pipeline Hazards]].
- **Bus** — shared communication pathway between CPU, memory, and devices. See [[Bus Architecture]].

### C
- **Cache coherence** — ensuring all cores see a consistent view of memory despite private caches. See [[Multicore and Multiprocessing]].
- **CISC (Complex Instruction Set Computer)** — ISA style with many, variable-length, complex instructions. See [[Instruction Set Architecture]].
- **CPI (Cycles Per Instruction)** — average clock cycles used per instruction executed. See [[Performance Metrics]].
- **Critical path** — the longest gate-delay chain in a combinational circuit, bounding clock frequency. See [[Combinational Logic Circuits]].

### D
- **Datapath** — the functional units and wiring that data flows through in a CPU. See [[CPU Datapath and Control Unit]].
- **DMA (Direct Memory Access)** — hardware that transfers data between device and memory without CPU intervention per byte. See [[IO Systems and Interrupts]].
- **DRAM (Dynamic RAM)** — main-memory technology storing bits as capacitor charge, requiring refresh. See [[Memory Technologies]].

### F
- **Flynn's Taxonomy** — classification of architectures by instruction/data stream multiplicity (SISD/SIMD/MISD/MIMD). See [[Flynns Taxonomy]].
- **Forwarding (bypassing)** — routing a result directly between pipeline stages to avoid a data-hazard stall. See [[Pipeline Hazards]].

### H
- **Harvard architecture** — separate instruction and data memories/buses. See [[Von Neumann vs Harvard Architecture]].
- **Hazard** — any condition preventing an instruction from proceeding as scheduled in a pipeline (structural, data, control). See [[Pipeline Hazards]].

### I
- **Interrupt** — asynchronous signal from hardware requesting CPU attention. See [[IO Systems and Interrupts]].
- **ISA (Instruction Set Architecture)** — the hardware/software contract of instructions, registers, and memory model. See [[Instruction Set Architecture]].

### L
- **Locality (temporal/spatial)** — the tendency of programs to reuse recently accessed data or access nearby data, underlying cache/memory hierarchy design. See [[Memory Hierarchy]].

### M
- **MESI protocol** — a cache-coherence protocol with Modified/Exclusive/Shared/Invalid line states. See [[Multicore and Multiprocessing]].
- **MMIO (Memory-Mapped I/O)** — mapping device registers into the CPU's normal address space. See [[Bus Architecture]].
- **Multiplexer (MUX)** — a circuit selecting one of several inputs based on select lines. See [[Combinational Logic Circuits]].

### O
- **Out-of-order execution** — executing ready instructions ahead of program order, retiring results in order. See [[Superscalar and Out-of-Order Execution]].

### P
- **Page fault** — trap triggered when accessing a virtual page not currently mapped to physical memory. See [[Virtual Memory]].
- **Pipelining** — overlapping execution of multiple instructions across stages to increase throughput. See [[Pipelining]].
- **Program Counter (PC)** — register holding the address of the current/next instruction. See [[CPU Datapath and Control Unit]].

### R
- **Register renaming** — mapping architectural registers to a larger physical register file to eliminate false dependencies. See [[Superscalar and Out-of-Order Execution]].
- **RISC (Reduced Instruction Set Computer)** — ISA style with a small set of simple, fixed-length, load/store instructions. See [[Instruction Set Architecture]].

### S
- **SIMD (Single Instruction, Multiple Data)** — one instruction applied across multiple data elements. See [[Flynns Taxonomy]].
- **SIMT (Single Instruction, Multiple Threads)** — GPU execution model where warps of threads execute in lockstep. See [[GPU Architecture]].
- **SRAM (Static RAM)** — fast memory technology built from flip-flop-like cells, used for registers/caches. See [[Memory Technologies]].
- **Superscalar** — issuing/executing multiple instructions per cycle via multiple execution units. See [[Superscalar and Out-of-Order Execution]].

### T
- **Two's complement** — the standard signed-integer representation used by virtually all CPUs. See [[Number Systems and Data Representation]].
- **TLB (Translation Lookaside Buffer)** — cache of recent virtual-to-physical address translations. See [[Virtual Memory]].

### V
- **Virtual memory** — hardware/OS mechanism giving each process an isolated, potentially oversized address space. See [[Virtual Memory]].
- **Von Neumann architecture** — unified instruction/data memory and bus. See [[Von Neumann vs Harvard Architecture]].
- **VLIW (Very Long Instruction Word)** — ISA style exposing ILP explicitly via compiler-packed wide instructions. See [[Instruction Set Architecture]].

## See also
- [[Computer Architecture MOC]]

#glossary #reference
