---
tags: [cpu, isa, addressing-modes]
---

# Addressing Modes

> [!summary] Summary
> An addressing mode defines how an instruction specifies the location of its operand(s). Richer addressing modes let compilers generate denser code (fewer instructions) at the cost of more complex decode/execute hardware — a core RISC/CISC design trade-off (see [[Instruction Set Architecture]]).

| Mode | Operand location | Example (generic asm) | Notes |
|---|---|---|---|
| Immediate | Encoded directly in the instruction | `ADDI R1, R2, #5` | Fastest — no memory/register lookup needed for the operand |
| Register | In a named register | `ADD R1, R2, R3` | Fast, no memory access |
| Direct (absolute) | Address given literally | `LOAD R1, 0x1000` | Simple but inflexible (address fixed at compile/assembly time) |
| Register indirect | Address held in a register | `LOAD R1, (R2)` | Base for pointers |
| Displacement (base+offset) | address = register + constant | `LOAD R1, 8(R2)` | Standard for stack frames, struct field access |
| Indexed | address = base register + index register (×scale) | `LOAD R1, (R2, R3, 4)` | Array element access: `base + i*size` |
| PC-relative | address = PC + offset | `BEQ label` (offset from current PC) | Used for branches → position-independent code |
| Autoincrement/decrement | register used as address, then incremented/decremented | `LOAD R1, (R2)+` | Efficient array traversal, stack push/pop |

> [!example] Why displacement addressing matters
> Accessing a struct field like `p->x` compiles almost directly to a single displacement-addressed load: `LOAD R1, offset_of_x(Rp)` — no separate address-arithmetic instruction needed. This is exactly the kind of "compiler convenience" that CISC ISAs push into hardware, while RISC ISAs would compute the offset separately then use a simple base+0 load — trading instruction count for hardware simplicity.

## Effective address calculation

The **effective address (EA)** is the actual memory address computed from an addressing mode. For indexed addressing:

$$ EA = \text{Base} + (\text{Index} \times \text{Scale}) + \text{Displacement} $$

This is exactly the address computation done by array-indexing code (`arr[i]` for element size `Scale`).

## PC-relative addressing and position independence

Branches and calls typically use PC-relative addressing so that code can be loaded at *any* base address in memory and still execute correctly without modification — essential for shared libraries and ASLR (address space layout randomization), a security mitigation. See [[Virtual Memory]] for how this interacts with process address spaces.

## See also
- [[Instruction Set Architecture]]
- [[CPU Datapath and Control Unit]]
- [[Virtual Memory]]

#cpu #isa #addressing-modes
