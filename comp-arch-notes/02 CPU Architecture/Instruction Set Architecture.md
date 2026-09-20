---
tags: [cpu, isa, risc, cisc]
---

# Instruction Set Architecture (ISA)

> [!summary] Summary
> The ISA is the contract between hardware and software: the set of instructions, registers, memory model, and data types a CPU exposes to programmers/compilers. It is the line that separates "architecture" (what software sees) from "microarchitecture" (how the hardware implements it — see [[CPU Datapath and Control Unit]], [[Pipelining]]).

## 1. What an ISA specifies
- Instruction formats (opcode + operand encoding)
- Register set (names, count, width)
- Addressing modes (see [[Addressing Modes]])
- Data types and sizes supported natively
- Memory model (byte-addressable? endianness? alignment rules?)
- Exception/interrupt behavior (see [[IO Systems and Interrupts]])

Examples: x86-64, ARMv8/ARMv9, RISC-V, MIPS, POWER.

## 2. RISC vs CISC

| | RISC (Reduced Instruction Set Computer) | CISC (Complex Instruction Set Computer) |
|---|---|---|
| Instruction count | Small, simple, uniform | Large, many specialized instructions |
| Instruction length | Fixed (e.g., always 32 bits) | Variable (1–15 bytes on x86) |
| Execution model | Load/store — only load/store instructions touch memory; everything else is register-to-register | Instructions can operate directly on memory operands |
| Cycles per instruction | Mostly 1 (simple, uniform pipeline) | Varies widely, some instructions take many cycles |
| Decode complexity | Simple, fast decode | Complex decode (variable length, many addressing modes) |
| Compiler responsibility | High — compiler does more optimization work | Lower — hardware/microcode absorbs complexity |
| Examples | RISC-V, ARM, MIPS, POWER, SPARC | x86/x86-64, VAX, System/360 |

> [!note] The RISC philosophy
> RISC emerged in the 1980s (Patterson & Hennessy at Berkeley, Hennessy at Stanford/MIPS) from the observation that complex CISC instructions were rarely used by compilers, yet consumed die area and slowed down decode for every instruction. Making instructions simple and uniform enabled effective [[Pipelining]] — one instruction fetched and decoded per cycle.

> [!warning] The RISC/CISC line has blurred
> Modern x86 CPUs internally **decode CISC instructions into RISC-like micro-ops (µops)** which then execute on a RISC-style superscalar, out-of-order core (see [[Superscalar and Out-of-Order Execution]]). So at the microarchitectural level, the distinction mostly disappears — it now mainly affects instruction *decode* complexity and code density, not core execution style.

## 3. Instruction formats

A generic instruction is `opcode | operand(s)`. RISC ISAs typically use one of a few fixed formats:

```
R-type (register-register):  | opcode | rs1 | rs2 | rd  | funct |
I-type (immediate):          | opcode | rs1 | rd  | immediate     |
S-type (store):              | opcode | rs1 | rs2 | immediate(split) |
B-type (branch):             | opcode | rs1 | rs2 | offset(split) |
J-type (jump):                | opcode | rd  | large immediate offset |
```
(shown schematically; exact bit widths differ per ISA — RISC-V's above are illustrative of the general RISC pattern)

CISC formats (e.g., x86) instead have a variable-length encoding: optional prefixes, an opcode byte (or multi-byte opcode), an optional ModR/M byte encoding addressing mode, optional SIB byte, optional displacement, optional immediate — hence 1–15 byte instructions.

## 4. Instruction categories

| Category | Examples | Purpose |
|---|---|---|
| Data movement | `LOAD`, `STORE`, `MOV` | Move data between registers/memory |
| Arithmetic/Logic | `ADD`, `SUB`, `AND`, `OR`, `SHL` | Computation (executed on the ALU, see [[Combinational Logic Circuits]]) |
| Control flow | `JMP`, `BEQ`, `CALL`, `RET` | Change PC — branches, function calls |
| System | `SYSCALL`, `TRAP`, `HALT` | Interact with OS/privileged mode |

## 5. Instruction Level Parallelism (ILP) exposure

Some ISAs expose parallelism explicitly:
- **VLIW (Very Long Instruction Word)**: the compiler packs multiple independent operations into one very-wide instruction word, executed together — hardware doesn't need to discover parallelism at runtime (used in Itanium, some DSPs). Contrast with [[Superscalar and Out-of-Order Execution]] where hardware discovers ILP dynamically.

## See also
- [[Addressing Modes]]
- [[CPU Datapath and Control Unit]]
- [[Pipelining]]
- [[Superscalar and Out-of-Order Execution]]

#cpu #isa
