---
tags: [rust, fundamentals, types]
---

# Data Types

> [!summary] Summary
> Rust is statically typed — every value's type is known at compile time, usually inferred rather than explicitly annotated. This note surveys the scalar and compound types built into the language.

## 1. Scalar types

| Category | Types | Notes |
|---|---|---|
| Integers | `i8`,`i16`,`i32`,`i64`,`i128`,`isize` (signed); `u8`,`u16`,`u32`,`u64`,`u128`,`usize` (unsigned) | `i32` is the default if unannotated and inferable; `isize`/`usize` match the pointer width of the target platform, used for indexing |
| Floating-point | `f32`, `f64` | `f64` is the default — same precision as most languages' native "double" |
| Boolean | `bool` | `true` / `false`, exactly one byte |
| Character | `char` | a full Unicode scalar value (4 bytes), NOT just ASCII — distinct from a single UTF-8 byte |

```rust
let x: i32 = -5;
let y = 3.14;          // inferred f64
let is_active = true;
let heart: char = '❤';  // a full unicode scalar, not one byte
```

> [!warning] Integer overflow behavior
> In debug builds, integer overflow (e.g., `u8` value 255 + 1) causes a **panic**. In release builds (optimized), it instead **wraps around** (two's complement wrapping) by default for performance reasons — a genuine behavioral difference between debug and release that has bitten developers who only tested in debug mode. Explicit methods (`checked_add`, `wrapping_add`, `saturating_add`, `overflowing_add`) let you choose the exact behavior you want regardless of build profile.

## 2. Compound types

### Tuples
Fixed-size, heterogeneous groupings:
```rust
let point: (i32, i32, &str) = (3, 4, "origin-relative");
let (x, y, label) = point;   // destructuring
println!("{}", point.0);      // access by index
```

### Arrays
Fixed-size, homogeneous, stored entirely on the stack (unlike [[Common Collections|Vec]], which is heap-allocated and growable):
```rust
let numbers: [i32; 5] = [1, 2, 3, 4, 5];
let zeros = [0; 100];   // 100 zeros, shorthand for repeated initialization
```

Array length is part of the type (`[i32; 5]` and `[i32; 6]` are different types) and is checked at compile time wherever the length is statically known — indexing out of bounds on a runtime-provided index is checked and panics at runtime rather than causing undefined behavior, unlike C.

## 3. Type inference

Rust infers types in the overwhelming majority of cases — explicit annotations are needed mainly at function signatures (always required) and in genuinely ambiguous contexts:

```rust
let v = Vec::new();      // ← type unknown yet
v.push(5);                // ← now inferred as Vec<i32> from usage
```

```rust
let guess: u32 = "42".parse().expect("not a number"); // annotation needed: parse() is generic
```

## 4. Type conversion

Rust has **no implicit numeric coercion** (unlike C/C++/JavaScript) — converting between numeric types always requires an explicit `as` cast or a fallible conversion method:

```rust
let a: i32 = 300;
let b = a as u8;              // explicit cast — truncates (b = 44), no warning
let c: u8 = a.try_into().unwrap_or_default(); // fallible, safer conversion — errors instead of silently truncating
```

> [!tip] Prefer TryFrom/TryInto over `as` for anything user-facing
> `as` casting silently truncates or wraps on overflow — fine for deliberate bit-level reinterpretation, risky for converting arbitrary runtime values. `TryFrom`/`TryInto` return a `Result` (see [[Option and Result]]) that surfaces overflow as an explicit, handleable error instead.

## See also
- [[Variables and Mutability]]
- [[Common Collections]]
- [[Option and Result]]

#rust #fundamentals #types
