---
tags: [fundamentals, digital-logic, boolean-algebra]
---

# Boolean Algebra and Logic Gates

> [!summary] Summary
> Boolean algebra is the mathematics of true/false logic. Logic gates are the physical (transistor-built) implementation of Boolean functions, and they are the atoms from which every digital circuit — ALUs, registers, entire CPUs — is built.

## 1. Boolean algebra basics

Variables take values `0` (false) and `1` (true). Core operators: AND (`·`), OR (`+`), NOT (`'` or overline).

### Fundamental laws

| Law | AND form | OR form |
|---|---|---|
| Identity | $A \cdot 1 = A$ | $A + 0 = A$ |
| Null | $A \cdot 0 = 0$ | $A + 1 = 1$ |
| Idempotent | $A \cdot A = A$ | $A + A = A$ |
| Complement | $A \cdot A' = 0$ | $A + A' = 1$ |
| Commutative | $AB = BA$ | $A+B=B+A$ |
| Associative | $(AB)C = A(BC)$ | $(A+B)+C = A+(B+C)$ |
| Distributive | $A(B+C) = AB+AC$ | $A+BC = (A+B)(A+C)$ |
| Absorption | $A + AB = A$ | $A(A+B) = A$ |

### De Morgan's Laws (extremely important)

$$ (A \cdot B)' = A' + B' \qquad (A + B)' = A' \cdot B' $$

> [!tip] Why De Morgan's matters
> It lets you convert any AND/OR/NOT circuit into an all-NAND or all-NOR circuit — both NAND and NOR are **functionally complete** (universal), which is why real chips are built almost entirely from NAND gates: fewer transistors, uniform fabrication.

## 2. Logic gate symbols and truth tables

![[logic-gates.svg]]

| Gate | Expression | 1 when... |
|---|---|---|
| AND | $Y=A\cdot B$ | both inputs are 1 |
| OR | $Y=A+B$ | at least one input is 1 |
| NOT | $Y=A'$ | input is 0 |
| NAND | $Y=(AB)'$ | not both inputs are 1 (universal gate) |
| NOR | $Y=(A+B)'$ | neither input is 1 (universal gate) |
| XOR | $Y=A\oplus B$ | inputs differ |
| XNOR | $Y=(A\oplus B)'$ | inputs are the same |

> [!example] XOR is the heart of addition
> A single-bit adder's *sum* output is exactly `A XOR B XOR Cin`. This is why XOR gates appear throughout ALUs — see [[CPU Datapath and Control Unit]].

## 3. Canonical (Sum-of-Products / Product-of-Sums) forms

Any Boolean function can be written as a **sum of minterms** (OR of AND terms), directly readable off a truth table by taking every row where output = 1.

> [!example] SOP from a truth table
> | A | B | C | F |
> |---|---|---|---|
> | 0 | 0 | 1 | 1 |
> | 0 | 1 | 1 | 1 |
> | 1 | 0 | 0 | 1 |
>
> $F = A'B'C + A'BC + AB'C'$

## 4. Karnaugh maps (K-maps)

A K-map groups adjacent 1s (in powers of 2: 1, 2, 4, 8…) to minimize a Boolean expression visually, exploiting the fact that adjacent cells differ in exactly one variable (Gray-code ordering).

```
        BC
        00  01  11  10
   A  0 | 0 | 1 | 1 | 0 |
      1 | 0 | 1 | 1 | 0 |
```
Grouping the middle two columns (BC=01, BC=11) for both rows of A gives the simplified term `C` alone — replacing a 4-term SOP with a single literal.

**Rules of thumb:**
- Groups must have size $2^n$ (1, 2, 4, 8, …).
- Bigger groups → simpler (fewer-literal) terms.
- Don't-care conditions (`X`) can be included in a group if convenient, ignored otherwise.

## 5. Functional completeness

A gate set is **functionally complete** if it can implement any Boolean function. `{AND, OR, NOT}` is complete; so is `{NAND}` alone; so is `{NOR}` alone. This single fact underlies why fabs can build entire CPUs from one repeated cell (a NAND gate) at the transistor level.

## See also
- [[Combinational Logic Circuits]] — building adders, muxes, decoders from gates
- [[Sequential Logic Circuits]] — adding memory (state) to logic
- [[Number Systems and Data Representation]] — what the bits going into these gates represent

#fundamentals #digital-logic
