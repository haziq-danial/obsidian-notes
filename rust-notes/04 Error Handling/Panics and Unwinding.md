---
tags: [rust, panics, error-handling]
---

# Panics and Unwinding

> [!summary] Summary
> A panic is Rust's response to an **unrecoverable** error — a bug the program has no sensible way to continue past. Unlike a `Result`'s explicit, recoverable error path (see [[Option and Result]]), a panic unwinds the stack (or aborts) and, by default, terminates the program.

## 1. What triggers a panic

- Explicit calls: `panic!("message")`, `.unwrap()`/`.expect()` on a `None`/`Err`.
- Runtime-checked violations: out-of-bounds array/slice indexing, integer overflow in debug builds (see [[Data Types]] §1), division by zero, and other conditions the language guarantees are checked rather than left as undefined behavior.

```rust
let v = vec![1, 2, 3];
v[99];   // panics at runtime: "index out of bounds: the len is 3 but the index is 99"
```

> [!note] Checked, not undefined
> This is a deliberate contrast with C, where an out-of-bounds array access is undefined behavior (potentially reading arbitrary memory, a serious security hazard). Rust's runtime bounds checks turn what would be a memory-safety vulnerability in C into a well-defined, controlled panic — a real cost in code size/CPU cycles for the check itself, but a categorically safer failure mode.

## 2. Unwinding vs aborting

By default, a panic **unwinds** the stack: Rust walks back up through each function call, running `Drop` implementations (see [[Deref and Drop Traits]]) to clean up resources along the way, before ultimately terminating the thread (or process, if it's the main thread).

```toml
# Cargo.toml — opt into abort instead of unwind
[profile.release]
panic = "abort"
```

`panic = "abort"` skips the unwinding process entirely, terminating immediately — produces a smaller binary and slightly faster panics, at the cost of losing the ability to `catch_unwind` (below) or run cleanup logic on the way out. Common for embedded targets where unwinding machinery isn't available or worth the binary size.

## 3. Catching panics (rare, specific use cases)

```rust
let result = std::panic::catch_unwind(|| {
    panic!("something went wrong");
});
// result is an Err here, program continues
```

`catch_unwind` exists mainly for isolating failures at a boundary — e.g., a web server catching a panic in one request handler so it doesn't take down the entire process, or FFI boundaries where a panic must not unwind across a foreign (e.g., C) call stack. It is **not** meant as a general substitute for `Result`-based error handling — reach for `Result` (see [[Option and Result]]) for any error condition you actually expect and want to handle programmatically.

## 4. When to panic vs when to return Result

> [!example] Decision guide
> - **Prototype/example code, tests**: `.unwrap()` is fine — you want failures to be loud and immediate.
> - **A function whose contract guarantees a value can't be invalid** (e.g., you just checked `if !v.is_empty()` immediately before indexing `v[0]`): `.unwrap()` or direct indexing is reasonable — the panic would only ever fire if there's a genuine logic bug.
> - **Library code exposed to callers who might pass bad input**: return `Result`, letting the caller decide how to handle or report the failure — a library that panics on any malformed input is hostile to its consumers.
> - **A violated invariant that indicates corrupted internal state**: panicking is often correct — continuing to run with broken invariants risks worse consequences (silent data corruption) than stopping immediately.

## 5. Panics and safety

A panic is always memory-safe — it never leaves the program in an inconsistent, partially-freed state, thanks to `Drop` running during unwinding. This is distinct from the C/C++ world, where responding to a fatal error incorrectly (or not at all) can leave memory corrupted. However, a panic **can** poison a `Mutex` (see [[Shared State Concurrency]]) if it happens while a lock is held, which is a real practical consideration in concurrent code.

## See also
- [[Option and Result]]
- [[The Question Mark Operator]]
- [[Deref and Drop Traits]]

#rust #panics #error-handling
