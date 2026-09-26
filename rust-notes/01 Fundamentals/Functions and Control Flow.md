---
tags: [rust, fundamentals, control-flow]
---

# Functions and Control Flow

> [!summary] Summary
> Rust's control flow constructs are largely familiar from C-family languages, but with one pervasive difference: nearly everything is an **expression** that produces a value, including `if`, `match`, and blocks themselves — a design that eliminates a lot of the boilerplate other languages need for the same logic.

## 1. Functions

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b   // no semicolon: this is the returned expression
}
```

- Parameter types are always required (no inference across function boundaries — this keeps type-checking local and comprehensible one function at a time).
- The **last expression** in a block (no trailing semicolon) is that block's value — this is how a function returns a value without an explicit `return`, though `return` is available for early exits.
- Adding a semicolon turns an expression into a statement, discarding its value (as `()`, the unit type) — a common source of "expected type X, found ()" compiler errors for beginners who accidentally add a stray semicolon on the final line.

## 2. if as an expression

```rust
let condition = true;
let number = if condition { 5 } else { 6 };   // if is an expression here
```

Both branches must produce the **same type**, since the compiler must know `number`'s type regardless of which branch executes — a compile error results from `if condition { 5 } else { "six" }`.

## 3. Loops

```rust
// loop: unconditional, runs until an explicit break
let result = loop {
    counter += 1;
    if counter == 10 {
        break counter * 2;   // loop can produce a value via break!
    }
};

// while: conditional
while counter != 0 {
    counter -= 1;
}

// for: the idiomatic way to iterate — see also Iterators and Closures
for element in [10, 20, 30] {
    println!("{element}");
}
for number in (1..4).rev() {   // range, reversed
    println!("{number}!");
}
```

> [!tip] Prefer `for` over manual indexing
> Iterating with `for x in collection` (or `.iter()`) avoids both the off-by-one errors possible with manual index loops and the (checked, but still a runtime panic) cost of out-of-bounds indexing — see [[Iterators and Closures]] for the fuller iterator story.

### Loop labels
Nested loops can be labeled to target `break`/`continue` at a specific level:
```rust
'outer: for x in 0..5 {
    for y in 0..5 {
        if y == 2 { continue 'outer; }
        if x == 3 { break 'outer; }
    }
}
```

## 4. match as an expression

`match` is Rust's most powerful control-flow construct — exhaustive pattern matching that itself produces a value (see [[Enums and Pattern Matching]] for the full treatment):

```rust
let description = match number {
    0 => "zero",
    1 | 2 => "one or two",       // multiple patterns
    3..=9 => "three through nine", // inclusive range pattern
    _ => "something else",        // required: match must be exhaustive
};
```

## 5. Blocks as expressions

Any `{ }` block evaluates to its final expression's value, which is what makes constructs like `if`/`match`/`loop` composable as expressions in the first place:

```rust
let y = {
    let x = 3;
    x + 1     // this block evaluates to 4
};
```

## See also
- [[Enums and Pattern Matching]]
- [[Iterators and Closures]]
- [[Data Types]]

#rust #fundamentals #control-flow
