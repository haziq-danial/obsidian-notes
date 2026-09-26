---
tags: [rust, testing]
---

# Testing in Rust

> [!summary] Summary
> Testing is a first-class, built-in part of Rust's tooling: `#[test]`-annotated functions run via `cargo test` with no external test framework required, and Rust's type system (particularly `Result`/`Option`, see [[Option and Result]]) already eliminates entire categories of bugs unit tests would otherwise need to cover in less strictly-typed languages.

## 1. Unit tests — colocated with the code they test

```rust
pub fn add(a: i32, b: i32) -> i32 { a + b }

#[cfg(test)]
mod tests {
    use super::*;   // bring the parent module's items into scope

    #[test]
    fn adds_two_numbers() {
        assert_eq!(add(2, 2), 4);
    }

    #[test]
    #[should_panic(expected = "divide by zero")]
    fn division_by_zero_panics() {
        divide(1, 0);
    }

    #[test]
    fn parsing_fails_gracefully() -> Result<(), String> {
        let parsed: i32 = "42".parse().map_err(|_| "parse failed".to_string())?;
        assert_eq!(parsed, 42);
        Ok(())    // tests can return Result, using ? just like ordinary functions
    }
}
```

`#[cfg(test)]` ensures the entire `tests` module (and any test-only helper code inside it) is compiled **only** when running `cargo test` — it adds zero size or overhead to the normal release binary.

### Key assertion macros
| Macro | Checks |
|---|---|
| `assert!(expr)` | `expr` is `true` |
| `assert_eq!(a, b)` / `assert_ne!(a, b)` | equality / inequality, printing both values on failure |
| `#[should_panic]` | the test function panics (optionally matching an `expected` substring) |

## 2. Integration tests — testing the public API from outside

```
my_project/
├── src/
│   └── lib.rs
└── tests/
    └── integration_test.rs
```

```rust
// tests/integration_test.rs
use my_project::add;

#[test]
fn it_adds_two() {
    assert_eq!(add(2, 2), 4);
}
```

Each file in `tests/` is compiled as its **own separate crate**, linked against your library exactly as an external consumer would use it — meaning integration tests can only call your crate's `pub` API, making them a natural check that your public interface actually works as intended for real callers, not just internally.

## 3. Running tests

```bash
cargo test                       # run all tests
cargo test adds_two               # run only tests whose name contains this substring
cargo test -- --test-threads=1    # force sequential execution (tests run in parallel by default)
cargo test -- --nocapture          # show println! output even for passing tests
cargo test --release               # run tests against optimized code (rarely needed, occasionally reveals different bugs)
```

> [!warning] Tests run in parallel by default — watch for shared state
> Since `cargo test` runs tests concurrently by default (using threads), tests that share mutable state (writing to the same file, reading a shared environment variable) can interfere with each other non-deterministically. Either avoid shared mutable state between tests, or run affected tests with `--test-threads=1`, or coordinate access explicitly (e.g., via a `Mutex`, see [[Shared State Concurrency]]).

## 4. Documentation tests

```rust
/// Adds two numbers together.
///
/// # Examples
///
/// ```
/// let result = my_project::add(2, 3);
/// assert_eq!(result, 5);
/// ```
pub fn add(a: i32, b: i32) -> i32 { a + b }
```

Code blocks inside doc comments are **compiled and run as tests** by `cargo test` — a uniquely Rust feature that keeps documentation examples honest: if the API changes and an example in a doc comment stops compiling or stops passing, `cargo test` fails immediately, rather than the documentation silently going stale.

## 5. Property-based and snapshot testing (ecosystem crates)

- **`proptest`** / **`quickcheck`**: generate many random inputs automatically and assert a property holds for all of them, rather than hand-writing individual example-based test cases — good for testing invariants over a wide input space.
- **`insta`**: snapshot testing — captures a value's serialized representation and diffs future runs against a saved "golden" snapshot, useful for catching unintended output changes in complex data structures.

## See also
- [[Cargo and the Rust Toolchain]]
- [[Option and Result]]
- [[Essential Crates]]

#rust #testing
