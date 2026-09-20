---
tags: [fundamentals, digital-logic, combinational]
---

# Combinational Logic Circuits

> [!summary] Summary
> Combinational circuits are built from logic gates with **no memory** — outputs depend only on current inputs. This note builds up the standard building blocks (adders, muxes, decoders, encoders, ALUs) used throughout [[CPU Datapath and Control Unit]].

## 1. Half adder and full adder

A **half adder** adds two bits, producing sum and carry:

$$ S = A \oplus B \qquad C_{out} = A \cdot B $$

A **full adder** also takes a carry-in, needed to chain multi-bit addition:

$$ S = A \oplus B \oplus C_{in} \qquad C_{out} = AB + C_{in}(A \oplus B) $$

```
        A   B  Cin | S  Cout
        0   0   0  | 0   0
        0   0   1  | 1   0
        0   1   0  | 1   0
        0   1   1  | 0   1
        1   0   0  | 1   0
        1   0   1  | 0   1
        1   1   0  | 0   1
        1   1   1  | 1   1
```

### Ripple-carry adder
Chain $n$ full adders, each `Cout` feeding the next `Cin`, to add n-bit numbers. Simple but slow: worst-case delay grows linearly with $n$ because the final carry can't be known until it has rippled through every stage.

> [!tip] Faster alternatives
> **Carry-lookahead adders** precompute *generate* ($G_i = A_iB_i$) and *propagate* ($P_i = A_i \oplus B_i$) signals for all bit positions in parallel, then compute all carries directly from these — trading extra gates for $O(\log n)$ delay instead of $O(n)$.

## 2. Multiplexers and demultiplexers

A **multiplexer (MUX)** selects one of $2^n$ inputs using $n$ select lines:

```
   I0 ─┐
   I1 ─┼─ [MUX] ── Y      Y = S'·I0 + S·I1   (2-to-1 mux)
        │
   S ───┘
```

MUXes implement conditional logic in hardware — e.g., choosing between an ALU result from a register vs. an immediate value (see the mux in [[CPU Datapath and Control Unit]]'s datapath diagram), or picking the next PC value between `PC+4` and a branch target.

A **demultiplexer** does the reverse: routes one input to one of $2^n$ outputs based on select lines.

## 3. Decoders and encoders

A **decoder** turns an n-bit binary code into one of $2^n$ active output lines (one-hot). Used for:
- Memory address decoding (selecting a specific row/word)
- Opcode decoding (turning an instruction's opcode field into control signals — see [[CPU Datapath and Control Unit]])

An **encoder** does the reverse: $2^n$ input lines → n-bit code. A **priority encoder** resolves multiple simultaneous inputs by picking the highest-priority active line — used in interrupt controllers (see [[IO Systems and Interrupts]]).

## 4. Comparators

A magnitude comparator determines whether $A>B$, $A=B$, or $A<B$ bit by bit, typically built from XNOR gates (equality per bit) combined with priority logic for the ordering bits.

## 5. The Arithmetic Logic Unit (ALU)

The ALU combines adders, comparators, and logic gates behind a **function-select mux** so a single unit can perform ADD, SUB, AND, OR, XOR, SLT (set-less-than), shifts, etc., chosen by a few control bits from the control unit.

```
        A ──┐
             ├─► [Adder]────┐
        B ──┤                │
             ├─► [AND]───────┼─► [MUX] ──► Result
             ├─► [OR]────────┤      ▲
             └─► [XOR]───────┘      │
                                ALUOp (select)
```

Subtraction reuses the adder: $A - B = A + (\overline{B} + 1)$ — invert B and set carry-in to 1 (two's complement negation, see [[Number Systems and Data Representation]]). This is precisely why two's complement is universal: one adder handles both operations.

## 6. Propagation delay and critical path

Every gate has a **propagation delay** (time for output to settle after inputs change). The **critical path** of a combinational circuit is its longest gate-delay chain, and it sets the maximum clock frequency for any sequential circuit built around it (see [[Sequential Logic Circuits]] and [[Pipelining]] — pipelining exists precisely to shorten the critical path per stage).

## See also
- [[Boolean Algebra and Logic Gates]] — the gates these circuits are built from
- [[Sequential Logic Circuits]] — adding state/memory
- [[CPU Datapath and Control Unit]] — how these blocks assemble into a CPU

#fundamentals #digital-logic #combinational-logic
