---
tags: [cpu, datapath, control-unit]
---

# CPU Datapath and Control Unit

> [!summary] Summary
> The **datapath** is the collection of functional units (registers, ALU, memories, muxes) and the wires connecting them that data flows through. The **control unit** generates the signals that steer that flow (which mux input to select, whether to write a register, etc.) based on the current instruction. Together they implement the ISA in hardware.

## 1. The fetch-decode-execute cycle

Every instruction goes through the same broad phases:

```mermaid
flowchart LR
    F[Fetch] --> D[Decode] --> E[Execute] --> M[Memory Access] --> W[Write Back]
    W --> F
```

1. **Fetch**: read the instruction at `[PC]` from instruction memory; compute `PC + 4` (next sequential instruction).
2. **Decode**: split the instruction into opcode + operand fields; read source registers from the register file; the control unit decodes the opcode into control signals.
3. **Execute**: the ALU computes a result (arithmetic, logic, or an effective address for a memory op, or a branch condition).
4. **Memory access**: for loads/stores only — read from or write to data memory.
5. **Write back**: write the result into the destination register.

## 2. Single-cycle datapath

![[cpu-datapath-single-cycle.svg]]

In a **single-cycle** implementation, every instruction completes all five phases within one (long) clock cycle. Simple to design and reason about, but wasteful: the clock period must be long enough for the *slowest* instruction (typically a load), so every instruction — even a simple register-register ADD — takes as long as the slowest one.

### Key datapath elements
- **PC (Program Counter)**: holds the address of the current instruction (see [[Sequential Logic Circuits]] for the register it's built from).
- **Instruction memory**: read-only from the datapath's perspective during execution.
- **Register file**: typically 2 read ports + 1 write port, so an instruction can read two source registers and write one destination register in a single cycle.
- **Sign/zero extend**: widens an instruction's immediate field to the datapath's word width (see [[Number Systems and Data Representation]] for sign extension).
- **ALU**: performs the arithmetic/logic operation, or computes a memory effective address ([[Addressing Modes]]), or evaluates a branch condition.
- **Data memory**: read for loads, written for stores.
- **Muxes**: select between alternative data sources (e.g., ALU's second operand: register value vs. immediate; write-back value: ALU result vs. memory data).

## 3. Multi-cycle datapath

Splits execution into multiple shorter cycles (one per phase), reusing the *same* ALU and memory port for different phases of different instructions. This means:
- Each instruction uses only as many cycles as it needs (an ADD might finish in 3 cycles; a LOAD needs all 5).
- The clock period only needs to cover the *single slowest phase*, not the whole instruction — much shorter cycle time.
- The control unit becomes an explicit **finite state machine** (see [[Sequential Logic Circuits]] §4), since control signals must change from cycle to cycle within the execution of one instruction.

## 4. The control unit

The control unit maps the instruction's opcode (and sometimes function/funct field) to the set of control signals that drive every mux and enable line in the datapath.

| Control signal | Effect |
|---|---|
| `RegWrite` | enable write to register file |
| `ALUSrc` | select ALU's 2nd operand: register vs immediate |
| `ALUOp` | select ALU function (add/sub/and/or/…) |
| `MemRead` / `MemWrite` | enable data memory read/write |
| `MemToReg` | select write-back value: ALU result vs memory data |
| `Branch` | qualifies whether to use the branch-target PC |
| `Jump` | force PC to a jump target |

> [!example] Control signals for `ADD R1, R2, R3`
> `RegWrite=1, ALUSrc=0 (use register), ALUOp=ADD, MemRead=0, MemWrite=0, MemToReg=0 (use ALU result), Branch=0`

> [!example] Control signals for `LOAD R1, 8(R2)`
> `RegWrite=1, ALUSrc=1 (use immediate 8), ALUOp=ADD (compute address), MemRead=1, MemWrite=0, MemToReg=1 (use memory data), Branch=0`

### Hardwired vs microprogrammed control

- **Hardwired control**: control signals generated directly by combinational logic (a decoder + boolean equations) from the opcode. Fast, but inflexible — changing/extending the ISA requires redesigning logic. Typical of RISC CPUs.
- **Microprogrammed control**: each instruction is decoded into a sequence of **micro-instructions** stored in a small, fast "control store" (microcode ROM), each micro-instruction directly specifying one cycle's control signals. Easier to design/debug/patch (microcode updates!) at some cost to speed. Historically associated with CISC (see [[Instruction Set Architecture]]); modern x86 CPUs still use a microcode layer to translate CISC instructions into internal RISC-like µops.

## See also
- [[Instruction Set Architecture]]
- [[Addressing Modes]]
- [[Pipelining]] — overlapping the fetch/decode/execute/memory/writeback stages across instructions
- [[Combinational Logic Circuits]] — the ALU
- [[Sequential Logic Circuits]] — registers and the control FSM

#cpu #datapath #control-unit
