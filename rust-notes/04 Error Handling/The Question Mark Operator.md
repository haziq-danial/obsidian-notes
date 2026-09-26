---
tags: [rust, error-handling, question-mark-operator]
---

# The Question Mark Operator

> [!summary] Summary
> The `?` operator eliminates the boilerplate of manually matching and propagating errors after every fallible operation — it's syntactic sugar for "if this is an `Err`/`None`, return it immediately from the current function; otherwise, unwrap the `Ok`/`Some` value and keep going."

## 1. Before and after

```rust
// Without ?
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

// With ?
fn read_username() -> Result<String, std::io::Error> {
    let mut f = File::open("user.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}

// Chained even further
fn read_username_chained() -> Result<String, std::io::Error> {
    let mut s = String::new();
    File::open("user.txt")?.read_to_string(&mut s)?;
    Ok(s)
}
```

`?` after a `Result`-returning expression: if it's `Ok(v)`, evaluates to `v` and execution continues; if it's `Err(e)`, immediately returns `Err(e.into())` from the enclosing function.

## 2. ? works on Option too

```rust
fn last_char_of_first_line(text: &str) -> Option<char> {
    text.lines().next()?.chars().last()
}
```

If `.next()` returns `None`, the whole function immediately returns `None`. `?` cannot be mixed between `Option` and `Result` in the same function — the enclosing function's return type must match the kind of `?` being used.

## 3. The requirement: ? needs a compatible enclosing return type

```rust
fn main() {
    let f = File::open("hello.txt")?;   // ← compile error: `?` can't be used in a function returning ()
}
```

`?` can only be used inside a function whose return type is `Result` (or `Option`, or another type implementing the relevant `FromResidual`/`Try` trait machinery). `main` itself can be written to return `Result<(), E>` specifically to allow `?` at the top level:

```rust
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let f = File::open("hello.txt")?;
    Ok(())
}
```

## 4. Automatic error conversion via From

`?` automatically converts the error type via `From::from()` when propagating, which is what lets a function return `?` from operations with *different* underlying error types, as long as they all convert into the function's declared error type:

```rust
#[derive(Debug)]
enum AppError {
    Io(std::io::Error),
    Parse(std::num::ParseIntError),
}

impl From<std::io::Error> for AppError {
    fn from(e: std::io::Error) -> Self { AppError::Io(e) }
}
impl From<std::num::ParseIntError> for AppError {
    fn from(e: std::num::ParseIntError) -> Self { AppError::Parse(e) }
}

fn read_number() -> Result<i32, AppError> {
    let text = std::fs::read_to_string("num.txt")?;   // io::Error auto-converts via From
    let n: i32 = text.trim().parse()?;                  // ParseIntError auto-converts via From
    Ok(n)
}
```

This `From`-based conversion is exactly why real-world Rust code commonly defines one unified application error enum (or reaches for a crate like `thiserror`/`anyhow`, see [[Essential Crates]]) — it lets `?` transparently propagate errors from many different underlying libraries through one consistent error type.

## See also
- [[Option and Result]]
- [[Panics and Unwinding]]
- [[Essential Crates]] — thiserror and anyhow

#rust #error-handling #question-mark
