---
tags: [go, fundamentals, types]
---

# Variables and Types

> [!summary] Summary
> Go is statically typed with type inference for local variables. This note covers declaration syntax, the built-in types, zero values, and type conversion — the last of which, unlike many languages, is never implicit.

## 1. Declaring variables

```go
var x int = 5          // explicit type
var y = 5              // inferred as int
z := 5                  // short declaration — infers int, only valid inside functions
var a, b, c int         // multiple variables, same type
var name, age = "Al", 30  // multiple variables, inferred types
```

`:=` is the idiomatic form inside function bodies; `var` is required at package scope (where `:=` isn't allowed at all) and is also commonly used when you want to state a type explicitly or declare a variable without initializing it yet.

## 2. Zero values — no uninitialized variables

> [!note] Go has no "uninitialized" state
> Every declared variable is automatically initialized to its type's **zero value** if no initializer is given — there is no equivalent of an uninitialized, garbage-containing variable as in C. This eliminates an entire class of undefined-behavior bugs at the cost of the programmer needing to know what "zero" means for each type.

| Type | Zero value |
|---|---|
| Numeric types (`int`, `float64`, …) | `0` |
| `bool` | `false` |
| `string` | `""` (empty string) |
| Pointers, slices, maps, channels, functions, interfaces | `nil` |
| Structs | every field set to its own zero value, recursively |

## 3. Basic types

| Category | Types |
|---|---|
| Boolean | `bool` |
| String | `string` (immutable, UTF-8 byte sequence) |
| Signed integers | `int8`, `int16`, `int32`, `int64`, `int` (platform word size, 32 or 64-bit) |
| Unsigned integers | `uint8` (alias `byte`), `uint16`, `uint32`, `uint64`, `uint` |
| Floating point | `float32`, `float64` |
| Complex | `complex64`, `complex128` |
| Rune | `rune` (alias for `int32`, represents a Unicode code point) |

> [!example] byte vs rune
> `byte` (an alias for `uint8`) represents one raw byte of data. `rune` (an alias for `int32`) represents one Unicode code point, which may be encoded as 1–4 bytes in UTF-8. Iterating a `string` with `for i, c := range s` yields **runes** (decoding UTF-8 as it goes) with `i` as the *byte* offset — indexing `s[i]` directly instead yields a raw **byte**, which is why naive byte-indexing into non-ASCII strings can split a multi-byte character.

## 4. Constants and iota

```go
const Pi = 3.14159
const (
    StatusOK       = 200
    StatusNotFound = 404
)

type Weekday int
const (
    Sunday Weekday = iota  // 0
    Monday                  // 1
    Tuesday                 // 2
    Wednesday               // 3
)
```

`iota` resets to 0 at each `const` block and increments by one per line — the idiomatic way to build simple enumerated constants, since Go has no dedicated `enum` keyword.

## 5. Type conversion is always explicit

```go
var i int = 42
var f float64 = float64(i)   // explicit conversion required
var u uint = uint(f)

var s string = string(rune(65))  // "A" — converts a rune's code point to its string representation
```

> [!warning] No implicit numeric conversion, even between "compatible" types
> Unlike C or JavaScript, Go never silently converts between numeric types, even between `int` and `int64`, or `int` and `float64` — every conversion is written explicitly with `T(v)`. This is a deliberate readability/safety choice: every place a value's representation changes is visible in the source, and truncating/overflowing conversions (e.g., `int8(300)`) can't happen by accident.

## 6. Type inference limits

Type inference in Go is purely local — it works for `:=` based on the initializer expression, and (since Go 1.18) for [[Generics|generic]] type parameters based on argument types, but Go never infers a function's parameter or return types, which must always be written explicitly. This is a deliberate contrast to languages with pervasive/global type inference: every function signature remains a complete, self-documenting contract.

## See also
- [[Control Flow]]
- [[Functions]]
- [[Arrays and Slices]]

#go #fundamentals #types
