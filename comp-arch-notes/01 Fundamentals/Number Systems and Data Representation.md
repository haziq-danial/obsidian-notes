---
tags: [fundamentals, number-systems, data-representation]
---

# Number Systems and Data Representation

> [!summary] Summary
> Computers store everything as bits. This note covers positional number systems, base conversion, signed-integer representations (two's complement), and floating-point (IEEE 754) — the foundation everything else in this vault sits on.

## 1. Positional number systems

A number in base $b$ with digits $d_n \ldots d_1 d_0$ has value:

$$ \sum_{i=0}^{n} d_i \times b^i $$

| Base | Name | Digits used |
|---|---|---|
| 2 | Binary | 0, 1 |
| 8 | Octal | 0–7 |
| 10 | Decimal | 0–9 |
| 16 | Hexadecimal | 0–9, A–F |

> [!example] Conversion example
> $2A7F_{16}$ = $2 \times 16^3 + 10 \times 16^2 + 7 \times 16^1 + 15 \times 16^0$
> = $8192 + 2560 + 112 + 15 = 10879_{10}$

### Binary ↔ Hex shortcut
Since $16 = 2^4$, group binary digits in fours from the right:
`1010 1111 0011` → `A F 3` → `0xAF3`

### Decimal → Binary (repeated division)
Divide by 2, record remainders bottom-up:
```
13 / 2 = 6 r 1
 6 / 2 = 3 r 0
 3 / 2 = 1 r 1
 1 / 2 = 0 r 1
→ 1101
```

## 2. Signed integer representations

| Scheme | How negatives are formed | Zero representations | Range (n bits) |
|---|---|---|---|
| Sign-and-magnitude | MSB = sign, rest = magnitude | +0 and −0 | $-(2^{n-1}-1)$ to $2^{n-1}-1$ |
| One's complement | invert all bits | +0 and −0 | $-(2^{n-1}-1)$ to $2^{n-1}-1$ |
| **Two's complement** | invert all bits, add 1 | single 0 | $-2^{n-1}$ to $2^{n-1}-1$ |

> [!note] Why two's complement won
> It has a single representation of zero, and addition/subtraction use the **same hardware adder** regardless of sign — no special-casing. This is why virtually every modern CPU uses it (see [[CPU Datapath and Control Unit]]).

**Computing two's complement of an n-bit number:** invert every bit, then add 1.

```
  0000 1101   (13)
  1111 0010   (invert)
+ 0000 0001
-----------
  1111 0011   (-13, two's complement, 8-bit)
```

**Sign extension**: to widen a two's complement number, replicate the sign bit into the new high-order positions. (`1101` → `1111 1101` for 4→8 bits, still −3.) This matters directly in [[CPU Datapath and Control Unit]] when immediates are sign-extended.

**Overflow rule**: adding two numbers of the *same sign* produces a *different* sign in the result → overflow. Adding numbers of different signs can never overflow.

## 3. Fixed-point vs floating-point

Fixed-point reserves a set number of bits for the fractional part — simple, but a limited/fixed dynamic range. Floating point (IEEE 754) instead stores a sign, exponent, and mantissa/significand, trading precision for enormous dynamic range.

### IEEE 754 single precision (32-bit)

| Sign | Exponent (8 bits, biased +127) | Mantissa (23 bits) |
|---|---|---|
| 1 bit | e.g. `10000010` | fractional part, implicit leading 1 |

$$ \text{value} = (-1)^{S} \times 1.\text{mantissa} \times 2^{(E - 127)} $$

> [!example] Encode −6.5 in IEEE-754 single precision
> 1. $6.5_{10} = 110.1_2 = 1.101 \times 2^2$
> 2. Sign = 1 (negative)
> 3. Exponent = $2 + 127 = 129 = 10000001_2$
> 4. Mantissa = `101` padded to 23 bits: `10100000000000000000000`
> 5. Result: `1 10000001 10100000000000000000000` = `0xC0D00000`

Double precision (64-bit) uses 1 sign bit, 11 exponent bits (bias 1023), 52 mantissa bits — same idea, more range/precision.

> [!warning] Floating-point pitfalls
> - Not all decimal fractions are exact in binary (0.1 + 0.2 ≠ 0.3 exactly) — this is a *representation* issue, not a CPU bug.
> - Special values: `Exponent = all 1s, mantissa = 0` → ±∞. `Exponent = all 1s, mantissa ≠ 0` → NaN. `Exponent = all 0s` → denormalized numbers (extend range near zero at reduced precision).

## 4. Character and byte-level encoding
- **ASCII**: 7-bit code for English characters/control codes (0–127).
- **Unicode / UTF-8**: variable-length encoding, backward-compatible with ASCII for bytes < 0x80.
- **Endianness**: the order bytes of a multi-byte word are stored in memory.
  - **Big-endian**: most significant byte at the lowest address (network byte order).
  - **Little-endian**: least significant byte at the lowest address (x86, most ARM configurations).

```
32-bit value 0x12345678 stored at address 0x00:

Big-endian:    [0x00]=12 [0x01]=34 [0x02]=56 [0x03]=78
Little-endian: [0x00]=78 [0x01]=56 [0x02]=34 [0x03]=12
```

## See also
- [[Boolean Algebra and Logic Gates]] — how bits are manipulated by hardware
- [[CPU Datapath and Control Unit]] — where sign extension and ALU arithmetic are used
- [[Memory Technologies]] — how bits are physically stored

#fundamentals #number-systems
