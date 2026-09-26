---
tags: [rust, ownership, core-concept]
---

# Ownership

> [!summary] Summary
> Ownership is Rust's central, defining feature: a compile-time-enforced set of rules governing how memory is managed, eliminating the need for a garbage collector while still preventing use-after-free, double-free, and dangling-pointer bugs. Every other Rust concept in this vault ultimately traces back to this system.

![[stack-vs-heap.svg]]

## 1. The three rules

1. Each value in Rust has a single **owner** (a variable).
2. There can only be **one owner at a time**.
3. When the owner goes out of scope, the value is **dropped** (its memory is freed).

These rules are checked entirely by the compiler — there is no runtime tracking, no reference counting by default, no garbage collection pause. The cost is paid once, at compile time, in the form of a stricter (and initially less familiar) set of constraints on how code can be written.

## 2. Move semantics

![[ownership-move-semantics.svg]]

For heap-allocated, non-`Copy` types (like `String`), assignment **moves** ownership rather than copying the data:

```rust
let s1 = String::from("hello");
let s2 = s1;              // s1 is MOVED into s2
println!("{}", s1);       // compile error: value borrowed after move
```

Only the stack-resident handle (pointer, length, capacity) is copied — the heap data itself is not duplicated. To prevent both `s1` and `s2` from later trying to free the same heap allocation (a **double free**), Rust considers `s1` invalid after the move and rejects any further use of it at compile time.

> [!example] Moves apply to function calls too
> ```rust
> fn takes_ownership(s: String) { println!("{s}"); }  // s dropped at end of this function
>
> let s = String::from("hi");
> takes_ownership(s);       // s is moved into the function
> // println!("{}", s);     // ← compile error: s no longer valid here
> ```
> Passing a non-`Copy` value to a function moves it, just like assignment does — this is exactly why functions that shouldn't consume their argument take a **reference** instead (see [[References and Borrowing]]).

## 3. The Copy trait — the exception to moving

Simple, fixed-size types that are cheap to duplicate implement `Copy`, which makes assignment **copy** the value instead of moving it — both bindings remain independently valid:

```rust
let x = 5;
let y = x;              // copy, not move
println!("{x} {y}");    // both valid — fine
```

Types that implement `Copy`: all the primitive scalar types (`i32`, `f64`, `bool`, `char`), and tuples/arrays composed entirely of `Copy` types. A type **cannot** implement both `Copy` and `Drop` — if a type needs custom cleanup logic on drop, it fundamentally cannot be "just duplicated" without also duplicating whatever that cleanup logic manages (e.g., you can't `Copy` a `String` because that would require two owners of the same heap buffer, defeating the entire purpose of ownership).

## 4. Clone — explicit, deep duplication

When you genuinely want a full independent copy of heap data (not just the stack handle), call `.clone()` explicitly:

```rust
let s1 = String::from("hello");
let s2 = s1.clone();     // deep copy — a genuinely new heap allocation
println!("{s1} {s2}");  // both valid, both own independent data
```

> [!tip] Why Clone is explicit, never implicit
> Because heap-allocating a full deep copy can be an expensive operation, Rust never does it silently — every `.clone()` call is visible in the source, so a reader (or the code's author) is never surprised by a hidden performance cost the way an implicitly-copying language might hide it.

## 5. Ownership and function return values

Ownership can be transferred back out of a function via its return value:

```rust
fn gives_ownership() -> String {
    String::from("yours now")   // moved out to the caller
}

fn takes_and_gives_back(s: String) -> String {
    s   // received ownership, then moved back out — same value, no copy
}
```

Passing ownership in and back out on every function that merely needs to *use* a value without keeping it would be tedious and inefficient for larger data — this exact friction is what [[References and Borrowing]] solves.

## See also
- [[References and Borrowing]]
- [[Lifetimes]]
- [[Deref and Drop Traits]]

#rust #ownership
