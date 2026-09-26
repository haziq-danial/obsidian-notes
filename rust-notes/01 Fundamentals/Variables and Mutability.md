---
tags: [rust, fundamentals, variables]
---

# Variables and Mutability

> [!summary] Summary
> Rust variables are **immutable by default** — a deliberate design choice that catches an entire class of accidental-mutation bugs at compile time and makes code easier to reason about, especially once concurrency enters the picture.

## 1. Immutable by default

```rust
let x = 5;
x = 6; // compile error: cannot assign twice to immutable variable
```

```rust
let mut x = 5;
x = 6; // fine — `mut` opts in to mutability
```

> [!note] Why default-immutable?
> In most other mainstream languages, mutability is the default and immutability is opt-in (`final`, `const`). Rust inverts this because in practice, most bindings never need to change after being set — defaulting to immutable makes the *exceptions* (places that do mutate) visible and deliberate, both to the reader and to the compiler's borrowing analysis (see [[References and Borrowing]]), which treats mutable and immutable bindings very differently.

## 2. Constants

```rust
const MAX_POINTS: u32 = 100_000;
```

- Always require an explicit type annotation.
- Must be a value computable at compile time (no function calls that aren't `const fn`).
- Can be declared in any scope, including global/module scope, unlike `let`.
- Are inlined at every usage site rather than occupying a fixed memory address (conceptually — the compiler is free to duplicate them wherever used).

## 3. Shadowing

Rust allows re-declaring a variable with the **same name** using `let` again, which creates an entirely new binding rather than mutating the old one:

```rust
let x = 5;
let x = x + 1;      // new binding, x is now 6
let x = x * 2;      // another new binding, x is now 12
{
    let x = "hi";   // shadows within this inner scope only
    println!("{}", x); // "hi"
}
println!("{}", x);     // 12 — outer x unaffected
```

> [!example] Shadowing vs mut — different tools for different intents
> ```rust
> let mut count = 0;
> count += 1;                     // mutation: same binding, same type, changing value
>
> let spaces = "   ";
> let spaces = spaces.len();      // shadowing: same name, but a DIFFERENT TYPE (&str → usize)
> ```
> `mut` changes a value in place, keeping the same type. Shadowing creates a fresh binding and can even change the *type* associated with a name — useful for a sequence of transformations on conceptually "the same" piece of data without inventing new names at each step (`guess`, `guess_trimmed`, `guess_parsed`, …).

## 4. Scope and drop order

A variable is valid from the point it's declared until the end of its enclosing scope (a `{ }` block, function body, etc.). When a scope ends, Rust calls `Drop` (see [[Deref and Drop Traits]]) on every owned value that goes out of scope, in reverse order of declaration — this is the mechanism underlying automatic memory management without a garbage collector, tied directly into [[Ownership]].

## See also
- [[Ownership]]
- [[Data Types]]
- [[Deref and Drop Traits]]

#rust #fundamentals #variables
