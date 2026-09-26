---
tags: [rust, fundamentals]
---

# What is Rust

> [!summary] Summary
> Rust is a systems programming language focused on performance, reliability, and memory safety — achieved entirely at **compile time**, with no garbage collector and no runtime overhead for its core safety guarantees. It was originally developed at Mozilla and is now stewarded by the independent Rust Foundation.

## 1. The core pitch

Rust's defining claim is resolving a tension that had long seemed fundamental in systems programming:

| | C / C++ | Garbage-collected languages (Go, Java, Python) | Rust |
|---|---|---|---|
| Manual memory control | Yes | No | Yes |
| Memory safety (no use-after-free, no data races) | No (programmer's responsibility) | Yes (via GC / runtime checks) | Yes (via compile-time checks) |
| Runtime overhead for safety | None | GC pauses, allocation overhead | None |

Rust achieves this via its **ownership system** (see [[Ownership]]), enforced by the compiler's **borrow checker** — a set of rules checked entirely during compilation, so a program that compiles is (for the classes of bugs Rust targets) guaranteed free of dangling pointers, use-after-free, double frees, and data races, without paying any cost for these checks while the program actually runs.

## 2. What Rust is used for

- **Systems programming**: operating system components, embedded systems, device drivers (traditionally C/C++ territory).
- **Command-line tools**: fast startup, single static binary, no runtime dependency (`ripgrep`, `fd`, `bat` are well-known Rust CLI tools).
- **WebAssembly**: Rust compiles cleanly to WASM, making it popular for performance-sensitive browser/edge code.
- **Backend services**: web servers and APIs (via frameworks like Axum, Actix), valued for predictable low-latency performance without a GC pause.
- **Blockchain and cryptography**: many blockchain projects (Solana, Polkadot) are written in Rust for its combination of performance and safety in security-critical code.

## 3. Key language properties

- **Zero-cost abstractions**: high-level constructs (iterators, generics, closures) compile down to code as efficient as hand-written low-level equivalents — you don't pay a runtime performance penalty for writing expressive code.
- **No null**: there is no null pointer in safe Rust; absence of a value is modeled explicitly via `Option<T>` (see [[Option and Result]]), eliminating an entire class of null-pointer-dereference bugs at compile time.
- **No exceptions**: recoverable errors are modeled as values (`Result<T, E>`) rather than thrown/caught exceptions — error handling becomes part of a function's type signature, visible at every call site (see [[Option and Result]]).
- **Expression-oriented**: most constructs (including `if`, `match`, and blocks) are expressions that produce a value, not just statements — enabling concise, functional-style code.
- **Pattern matching**: `match` and destructuring are woven throughout the language (see [[Enums and Pattern Matching]]).
- **Trait-based polymorphism**: shared behavior across types is expressed via traits rather than class inheritance (see [[Traits]]).

## 4. The compiler as a teacher

> [!tip] rustc's error messages are part of the language's design
> Rust's compiler is famous for unusually detailed, actionable error messages — often naming the exact rule violated (e.g., "cannot borrow `x` as mutable because it is also borrowed as immutable") and suggesting a fix. This isn't incidental: since Rust pushes so many checks (ownership, borrowing, lifetimes, exhaustive pattern matching) to compile time, the compiler's ability to explain *why* code is rejected is essential to the language actually being usable in practice, not just theoretically safe.

## 5. History in brief

Rust began as a personal project by Graydon Hoare in 2006, was sponsored by Mozilla starting in 2009, and reached a stable 1.0 release in 2015. Ownership has since passed to the independent **Rust Foundation** (2021), with major backing from companies including Amazon, Google, Microsoft, and Meta — reflecting Rust's growing adoption for security- and performance-critical infrastructure (e.g., parts of the Linux kernel, Windows, and Android now accept Rust code alongside C/C++).

## See also
- [[Cargo and the Rust Toolchain]]
- [[Ownership]]
- [[Variables and Mutability]]

#rust #fundamentals
