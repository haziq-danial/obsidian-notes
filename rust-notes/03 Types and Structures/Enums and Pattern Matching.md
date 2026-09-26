---
tags: [rust, enums, pattern-matching]
---

# Enums and Pattern Matching

> [!summary] Summary
> A Rust enum defines a type that can be **one of several named variants**, each optionally carrying its own data — far more expressive than enums in most other languages. Combined with exhaustive `match`, this lets the compiler guarantee every possible case is handled.

![[enum-pattern-matching.svg]]

## 1. Defining enums

```rust
enum Shape {
    Circle(f64),                          // tuple-style variant
    Rectangle { width: f64, height: f64 }, // struct-style variant
    Triangle(f64, f64, f64),
    Point,                                  // unit variant, no data
}
```

Each variant can carry different, independent data — something a C-style enum (just a set of named integers) or a plain tagged union in most languages can't express nearly as cleanly, since Rust's enum bundles the tag and its associated data into one type-checked unit.

## 2. match — exhaustive by construction

```rust
fn area(shape: &Shape) -> f64 {
    match shape {
        Shape::Circle(radius) => std::f64::consts::PI * radius * radius,
        Shape::Rectangle { width, height } => width * height,
        Shape::Triangle(a, b, c) => {
            let s = (a + b + c) / 2.0;
            (s * (s - a) * (s - b) * (s - c)).sqrt()
        }
        Shape::Point => 0.0,
        // omitting any variant here is a COMPILE ERROR
    }
}
```

> [!tip] Exhaustiveness as a refactoring safety net
> Adding a new variant to `Shape` later will cause a **compile error** at every existing `match` on `Shape` that doesn't handle it — the compiler finds every place that needs updating for you. This turns "did I remember to handle the new case everywhere?" from a runtime bug risk into a compile-time checklist, a significant advantage as codebases grow.

## 3. Pattern matching features

```rust
match number {
    0 => println!("zero"),
    1 | 2 => println!("one or two"),          // OR patterns
    3..=9 => println!("three through nine"),    // inclusive range
    n if n < 0 => println!("negative: {n}"),    // match guard
    n => println!("other: {n}"),                 // binds the value to `n`
}
```

```rust
// destructuring in a match arm
match shape {
    Shape::Rectangle { width, height } if width == height => println!("a square!"),
    Shape::Rectangle { width, height } => println!("{width} x {height}"),
    _ => {}
}
```

`_` matches anything without binding it — the catch-all pattern, required whenever a `match` doesn't otherwise cover every possibility (or when intentionally ignoring remaining cases).

## 4. if let and while let — matching a single pattern

When only one pattern actually matters and a full `match` would be overkill:

```rust
let config_max: Option<u8> = Some(3);
if let Some(max) = config_max {
    println!("max is {max}");
} else {
    println!("no max configured");
}
```

```rust
let mut stack = vec![1, 2, 3];
while let Some(top) = stack.pop() {
    println!("{top}");
}
```

## 5. Option and Result are just enums

Rust's most important types for handling absence and failure (see [[Option and Result]]) are themselves ordinary enums defined in the standard library:

```rust
enum Option<T> {
    Some(T),
    None,
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

There's nothing magic about them beyond being generic and receiving special ergonomic support (like the `?` operator, see [[The Question Mark Operator]]) — understanding enums and `match` is understanding the mechanism these central types are built on.

## 6. Enums with methods

Like structs, enums can have `impl` blocks:

```rust
impl Shape {
    fn describe(&self) -> String {
        match self {
            Shape::Circle(_) => "a circle".to_string(),
            Shape::Rectangle { .. } => "a rectangle".to_string(),
            _ => "some shape".to_string(),
        }
    }
}
```

## See also
- [[Structs]]
- [[Option and Result]]
- [[Traits]]

#rust #enums #pattern-matching
