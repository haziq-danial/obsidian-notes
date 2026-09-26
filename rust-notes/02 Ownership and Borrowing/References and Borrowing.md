---
tags: [rust, borrowing, references]
---

# References and Borrowing

> [!summary] Summary
> A reference lets you access a value **without taking ownership** of it — "borrowing" it temporarily. The borrow checker enforces two simple rules that together make data races on shared data impossible to express in safe Rust.

![[borrowing-rules.svg]]

## 1. Why borrowing exists

Without references, using a value in a function while keeping it available afterward would require passing ownership in and getting it back out every time (see [[Ownership]] §5) — workable but tedious. A reference lets a function borrow access temporarily:

```rust
fn calculate_length(s: &String) -> usize {   // borrows, doesn't take ownership
    s.len()
}

let s1 = String::from("hello");
let len = calculate_length(&s1);   // pass a reference
println!("{s1} has length {len}"); // s1 still valid here — it was never moved
```

## 2. The borrowing rules

At any given point, for a particular piece of data, Rust allows **either**:
- any number of **shared references** (`&T`) — read-only access from multiple places simultaneously, **or**
- exactly **one exclusive reference** (`&mut T`) — sole read/write access.

Never both at the same time. This single rule statically rules out data races: a data race requires two or more pointers accessing the same data with at least one doing a write and no synchronization — exactly the pattern this rule forbids by construction.

```rust
let mut s = String::from("hello");

let r1 = &s;          // ok — shared borrow
let r2 = &s;          // ok — another shared borrow
println!("{r1} {r2}");

let r3 = &mut s;      // ok — r1, r2's last use was above (non-lexical lifetimes)
r3.push_str(", world");
```

```rust
let mut s = String::from("hello");
let r1 = &s;
let r2 = &mut s;      // ← compile error: cannot borrow as mutable while borrowed as immutable
println!("{r1}");
```

> [!note] Non-lexical lifetimes (NLL)
> Modern Rust's borrow checker considers a reference's scope to end at its **last actual use**, not at the end of its enclosing block — this is why the first example above compiles: `r1`/`r2`'s last use is the `println!`, so by the time `r3` is created, they're no longer considered "active" borrows. This significantly loosened what earlier Rust compilers would reject compared to a naive lexical-scope reading of the rules.

## 3. Dangling references are impossible

```rust
fn dangle() -> &String {      // ← compile error
    let s = String::from("hello");
    &s
}   // s is dropped here — the reference would point to freed memory
```

The compiler rejects this outright: it can see that `s` is dropped at the end of `dangle`, so any reference to it would dangle. The fix is to return the owned `String` itself (transferring ownership out, per [[Ownership]] §5), not a reference to a local that's about to be destroyed.

## 4. References vs the values they borrow

A reference is itself a value (a pointer, essentially) with its own type (`&T` or `&mut T`), distinct from `T`. Dereferencing (`*r`) accesses the underlying value. Rust's **auto-deref** for method calls (`r.len()` instead of requiring `(*r).len()`) hides most of this ceremony in everyday code.

## See also
- [[Ownership]]
- [[Lifetimes]]
- [[Slices]]

#rust #borrowing #references
