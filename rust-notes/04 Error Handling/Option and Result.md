---
tags: [rust, error-handling, option, result]
---

# Option and Result

> [!summary] Summary
> Rust has no null and no exceptions. Instead, the possibility of "no value" is modeled by `Option<T>`, and the possibility of failure is modeled by `Result<T, E>` — both ordinary enums (see [[Enums and Pattern Matching]]), which means the compiler forces every caller to explicitly acknowledge and handle both cases.

## 1. Option<T> — modeling absence

```rust
enum Option<T> {
    Some(T),
    None,
}
```

```rust
fn find_user(id: u32) -> Option<String> {
    if id == 1 { Some(String::from("Alice")) } else { None }
}

match find_user(1) {
    Some(name) => println!("Found: {name}"),
    None => println!("No user with that id"),
}
```

> [!note] Why this eliminates null-pointer-style bugs
> In languages with null, *any* reference-typed value can silently be null, and forgetting to check produces a runtime crash (the infamous "billion-dollar mistake," per Tony Hoare, its inventor). In Rust, a plain `T` can **never** be absent — only an `Option<T>` can be `None`, and the compiler forces you to handle that case (via `match`, `if let`, or an explicit `.unwrap()`/`.expect()` that documents you're choosing to panic) before you can get at the `T` inside. The type system itself distinguishes "definitely present" from "maybe absent."

### Common Option methods

| Method | Effect |
|---|---|
| `.unwrap()` | get the value, **panic** if `None` — use only when `None` is truly impossible or acceptable to crash on |
| `.expect("message")` | like `.unwrap()`, but with a custom panic message — better for documenting *why* `None` shouldn't happen |
| `.unwrap_or(default)` | get the value, or a fallback default if `None` |
| `.unwrap_or_else(\|\| ...)` | like above, but the default is computed lazily |
| `.map(\|x\| ...)` | transform the inner value if `Some`, otherwise stays `None` |
| `.and_then(\|x\| ...)` | like `.map`, but the closure itself returns an `Option` — chains fallible steps |
| `.is_some()` / `.is_none()` | boolean checks without consuming the value |

## 2. Result<T, E> — modeling recoverable failure

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

```rust
use std::fs::File;

fn open_config() -> Result<File, std::io::Error> {
    File::open("config.toml")
}

match open_config() {
    Ok(file) => println!("Opened: {file:?}"),
    Err(e) => println!("Failed to open: {e}"),
}
```

Unlike exceptions in other languages, a `Result`-returning function's signature makes the possibility of failure **visible at every call site** — you can see from the type alone that a function might fail, without needing to check documentation or hope a try/catch is in the right place somewhere up the call stack.

### Common Result methods

| Method | Effect |
|---|---|
| `.unwrap()` | get the `Ok` value, **panic** with the `Err` value's debug output if `Err` |
| `.expect("message")` | like `.unwrap()`, with a custom panic message |
| `.unwrap_or(default)` | get the `Ok` value, or a fallback if `Err` |
| `.map(\|x\| ...)` | transform the `Ok` value, pass `Err` through unchanged |
| `.map_err(\|e\| ...)` | transform the `Err` value, pass `Ok` through unchanged — useful for converting between error types |
| `.ok()` | convert to `Option<T>`, discarding error details |

## 3. Choosing between panic and Result

> [!tip] The general guideline
> Use `panic!` (via `.unwrap()`/`.expect()` or directly) for situations that indicate a **bug** in the program itself — a broken invariant that should never happen if the code is correct. Use `Result` for failures that are **expected, recoverable conditions** the caller should decide how to handle — a file that might not exist, a network request that might time out, user input that might be malformed. See [[Panics and Unwinding]] for the full treatment of when panicking is appropriate.

## 4. Chaining fallible operations

Manually matching on every intermediate `Result` gets verbose fast:

```rust
fn read_username_verbose() -> Result<String, std::io::Error> {
    let f = File::open("user.txt");
    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };
    let mut s = String::new();
    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
```

The `?` operator exists specifically to eliminate this boilerplate — see [[The Question Mark Operator]] for the equivalent, much shorter version of this exact function.

## See also
- [[Enums and Pattern Matching]]
- [[The Question Mark Operator]]
- [[Panics and Unwinding]]

#rust #error-handling
