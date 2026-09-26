---
tags: [rust, glossary, reference]
---

# Rust Glossary

> [!summary] Summary
> Quick-reference definitions for terms used throughout the vault. Each term links to the note where it's covered in depth.

### A
- **async/await** — syntax for writing non-blocking, concurrent code compiled into a state machine (a Future). See [[Async Await]].
- **Arc** — atomic reference-counted smart pointer; the thread-safe counterpart to Rc. See [[Box RC and Interior Mutability]], [[Shared State Concurrency]].
- **Associated function** — a function defined in an `impl` block that doesn't take `self` (e.g., a constructor). See [[Structs]].

### B
- **Borrow checker** — the compiler component enforcing ownership and borrowing rules at compile time. See [[Ownership]], [[References and Borrowing]].
- **Box** — the simplest smart pointer; puts a value on the heap with a single owner. See [[Box RC and Interior Mutability]].

### C
- **Cargo** — Rust's official build tool and package manager. See [[Cargo and the Rust Toolchain]].
- **Closure** — an anonymous function that can capture variables from its enclosing scope. See [[Iterators and Closures]].
- **Copy** — a trait marking types that are duplicated by simple bitwise copy instead of moved. See [[Ownership]].
- **Crate** — the smallest unit of compilation: a binary or library. See [[Modules and Crates]].

### D
- **Deref coercion** — automatic conversion (e.g., `&String` → `&str`) enabled by implementing the Deref trait. See [[Deref and Drop Traits]].
- **Drop** — the trait defining cleanup logic run automatically when a value goes out of scope. See [[Deref and Drop Traits]].
- **Dynamic dispatch** — resolving which method implementation to call at runtime via a vtable, used by `dyn Trait`. See [[Trait Objects and Dynamic Dispatch]].

### E
- **Enum** — a type that can be one of several named variants, each optionally carrying data. See [[Enums and Pattern Matching]].
- **Executor** — the runtime component that polls futures to drive async tasks to completion. See [[Async Await]].

### F
- **Future** — a value representing an asynchronous computation that may not have completed yet. See [[Async Await]].
- **Fearless concurrency** — Rust's marketing term for compile-time-enforced thread safety. See [[Threads and Message Passing]].

### L
- **Lifetime** — the compiler's tracking of how long a reference remains valid. See [[Lifetimes]].
- **Lifetime elision** — the rules letting most function signatures omit explicit lifetime annotations. See [[Lifetimes]].

### M
- **match** — Rust's exhaustive pattern-matching control-flow construct. See [[Enums and Pattern Matching]].
- **mpsc** — "multiple producer, single consumer" — the standard channel type for message-passing concurrency. See [[Threads and Message Passing]].
- **Monomorphization** — generating a separate specialized function/type per concrete type used with a generic, at compile time. See [[Generics]].
- **Move** — transferring ownership of a value, invalidating the original binding. See [[Ownership]].
- **Mutex** — a mutual-exclusion lock; in Rust, wraps the data it protects directly. See [[Shared State Concurrency]].

### O
- **Option** — an enum modeling the presence (`Some`) or absence (`None`) of a value, replacing null. See [[Option and Result]].
- **Ownership** — Rust's core memory-management system: each value has exactly one owner. See [[Ownership]].
- **Orphan rule** — the restriction that a trait can only be implemented for a type if you own the trait or the type. See [[Traits]].

### P
- **Panic** — Rust's response to an unrecoverable error; unwinds (or aborts) the stack. See [[Panics and Unwinding]].
- **Procedural macro** — a macro implemented as Rust code operating on syntax trees (derive, attribute, or function-like). See [[Macros]].

### Q
- **? operator** — shorthand for propagating an `Err`/`None` immediately from the current function. See [[The Question Mark Operator]].

### R
- **Rc** — reference-counted smart pointer enabling multiple owners (single-threaded only). See [[Box RC and Interior Mutability]].
- **RefCell** — a type providing interior mutability with borrow rules checked at runtime instead of compile time. See [[Box RC and Interior Mutability]].
- **Result** — an enum modeling success (`Ok`) or failure (`Err`) of a fallible operation. See [[Option and Result]].

### S
- **Send** — a marker trait indicating a type is safe to transfer ownership of across threads. See [[Threads and Message Passing]].
- **Shadowing** — re-declaring a variable with the same name via `let`, creating a new binding. See [[Variables and Mutability]].
- **Slice** — a reference to a contiguous sequence within a collection, without owning it. See [[Slices]].
- **Static dispatch** — resolving which function implementation to call at compile time via monomorphization. See [[Generics]].
- **Sync** — a marker trait indicating a type is safe to share (`&T`) across threads. See [[Threads and Message Passing]].

### T
- **Trait** — a definition of shared behavior that types can implement; Rust's mechanism for polymorphism. See [[Traits]].
- **Trait object** — `dyn Trait`; a fat pointer enabling runtime polymorphism across different concrete types. See [[Trait Objects and Dynamic Dispatch]].

### U
- **Unsafe** — a block or function that opts out of a specific, small set of compiler safety checks. See [[Unsafe Rust]].

### V
- **Vec** — a growable, heap-allocated array, Rust's default general-purpose collection. See [[Common Collections]].
- **Vtable** — a virtual method table used to resolve dynamic dispatch calls at runtime. See [[Trait Objects and Dynamic Dispatch]].

## See also
- [[Rust MOC]]

#glossary #reference
