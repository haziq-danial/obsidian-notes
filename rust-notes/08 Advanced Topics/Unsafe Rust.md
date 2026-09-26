---
tags: [rust, unsafe]
---

# Unsafe Rust

> [!summary] Summary
> `unsafe` opts out of a small, specific set of the compiler's static safety checks, unlocking operations the compiler cannot otherwise verify are safe. It is not a wholesale "turn off the type system" switch — most of Rust's guarantees (ownership, type safety, no null) still fully apply inside `unsafe` blocks.

## 1. What unsafe actually unlocks — exactly five things

```rust
unsafe {
    // 1. Dereference a raw pointer
    // 2. Call an unsafe function or method
    // 3. Access or modify a mutable static variable
    // 4. Implement an unsafe trait
    // 5. Access fields of a union
}
```

That's the complete list. Everything else — the borrow checker's rules on ordinary references, move semantics, type checking — remains fully in force even inside an `unsafe` block. `unsafe` narrowly expands *what operations are allowed to be written*, not a general suspension of Rust's rules.

## 2. Raw pointers

```rust
let mut num = 5;
let r1 = &num as *const i32;         // immutable raw pointer
let r2 = &mut num as *mut i32;        // mutable raw pointer

unsafe {
    println!("r1 is: {}", *r1);        // dereferencing requires unsafe
    *r2 = 10;
}
```

Raw pointers (`*const T`, `*mut T`), unlike references, can be null, dangling, unaligned, and can have both a `*const` and `*mut` pointer to the same location simultaneously — none of the guarantees `&T`/`&mut T` provide apply. Creating a raw pointer is safe (it's just an integer-like value at that point); **dereferencing** one is where the compiler can no longer verify safety, hence `unsafe`.

## 3. Calling unsafe functions, and FFI

```rust
unsafe fn dangerous() { /* ... */ }
unsafe { dangerous(); }
```

The most common real-world use of `unsafe` is calling into **foreign code** (C libraries) via FFI, where Rust's compiler has no way to verify the foreign function's safety contract:

```rust
extern "C" {
    fn abs(input: i32) -> i32;
}

unsafe {
    println!("Absolute value of -3 via C: {}", abs(-3));
}
```

## 4. Why unsafe exists at all

> [!note] Unsafe is not a design flaw — it's a controlled escape hatch
> Some genuinely necessary operations (talking to hardware, calling C libraries, implementing the very lowest-level building blocks like `Vec` or `Mutex` themselves) cannot be proven safe by any static analysis, yet can absolutely be written correctly by a careful programmer who understands invariants the compiler can't see. `unsafe` exists so these operations remain possible, isolated to explicitly marked regions, rather than either being impossible in Rust at all, or requiring the *entire* language to abandon its safety guarantees.

Critically: nearly all of the standard library's own collections (`Vec`, `String`, `HashMap`) and synchronization primitives are themselves implemented using `unsafe` internally, then expose a completely safe API on top — `unsafe` is the tool used to *build* safe abstractions, not a signal that safety has been abandoned for consumers of that abstraction.

## 5. The safety contract

Writing `unsafe` code means the **programmer**, not the compiler, is now responsible for upholding Rust's invariants (no data races, no dangling references, valid memory access, etc.) for that specific block. Getting it wrong reintroduces exactly the bugs (use-after-free, buffer overflows, undefined behavior) safe Rust is designed to prevent — `unsafe` code that violates these invariants is a genuine bug, full stop, even though the compiler didn't catch it.

> [!warning] "Unsafe" doesn't mean "incorrect," but it does mean "unverified"
> A correct `unsafe` block, wrapped in a safe public API whose safety the author has manually verified, is normal and common in low-level Rust code (see any collection or concurrency primitive in the standard library). The concern isn't using `unsafe` at all — it's using it without a clear, correct justification for why the invariants the compiler would otherwise check are actually upheld.

## 6. Tools for auditing unsafe code

- **Miri**: an interpreter for Rust's mid-level IR that can detect many forms of undefined behavior (out-of-bounds access, use-after-free, invalid pointer arithmetic) in `unsafe` code that would otherwise compile and run "successfully" while secretly being UB.
- **`#![forbid(unsafe_code)]`**: a crate-level lint that makes any `unsafe` usage a hard compile error — useful for application code that has no legitimate need for it, keeping the "trusted" surface area at zero.

## See also
- [[Ownership]]
- [[References and Borrowing]]
- [[Modules and Crates]]

#rust #unsafe
