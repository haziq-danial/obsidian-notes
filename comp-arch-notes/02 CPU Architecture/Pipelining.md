---
tags: [cpu, pipelining, performance]
---

# Pipelining

> [!summary] Summary
> Pipelining overlaps the execution of multiple instructions, each in a different stage, the same way a car assembly line overlaps different cars at different stations. It doesn't reduce any single instruction's latency — it dramatically increases *throughput*.

## 1. The classic 5-stage pipeline

![[pipeline-5-stage.svg]]

| Stage | Name | What happens |
|---|---|---|
| IF | Instruction Fetch | Read instruction from instruction memory at `[PC]` |
| ID | Instruction Decode / Register Read | Decode opcode, read source registers, generate control signals |
| EX | Execute | ALU computes result or effective address; branch condition evaluated |
| MEM | Memory Access | Load/store to data memory (no-op for non-memory instructions) |
| WB | Write Back | Write result into destination register |

Each stage is separated by a **pipeline register** (see [[Sequential Logic Circuits]]) that latches that stage's outputs for the next stage to consume on the following clock edge.

## 2. Why pipelining works: throughput vs latency

For $n$ instructions through a $k$-stage pipeline (ideal, no stalls):

$$ \text{Total cycles} = k + (n - 1) $$

compared to $n \times k$ cycles if instructions ran fully sequentially with no overlap. As $n \to \infty$, throughput approaches **one instruction retired per cycle** (CPI → 1), even though any single instruction still takes $k$ cycles from fetch to writeback (latency is unchanged, or even slightly worse due to pipeline register overhead).

> [!note] Pipelining and clock frequency
> Splitting a single-cycle datapath into $k$ pipeline stages also shortens the combinational logic *per stage*, which — per the setup-time equation in [[Sequential Logic Circuits]] §5 — allows a **higher clock frequency**. This is the second, often larger, benefit of pipelining beyond simple overlap.

## 3. Pipeline depth trade-off

Deeper pipelines (more, shorter stages) allow higher clock frequency but:
- Increase the branch misprediction penalty (more stages to flush — see [[Pipeline Hazards]]).
- Increase the number of in-flight instructions the hardware must track (more hazard-detection/forwarding logic).
- Hit diminishing returns as per-stage logic becomes dominated by pipeline-register overhead rather than useful work.

The infamous Intel Pentium 4 pushed to 20–31 pipeline stages chasing clock frequency, but suffered badly on mispredicted branches — a cautionary tale that informed the shorter, wider pipelines of later designs (see [[Superscalar and Out-of-Order Execution]]).

## 4. Ideal vs actual CPI

$$ \text{CPI}_{actual} = \text{CPI}_{ideal} + \text{stall cycles per instruction} $$

Stalls come from **hazards** — see [[Pipeline Hazards]] for the full treatment of structural, data, and control hazards and how forwarding, stalling, and branch prediction address them.

## See also
- [[Pipeline Hazards]]
- [[Superscalar and Out-of-Order Execution]]
- [[CPU Datapath and Control Unit]]
- [[Performance Metrics]] — CPI, throughput, and how pipelining affects them

#cpu #pipelining
