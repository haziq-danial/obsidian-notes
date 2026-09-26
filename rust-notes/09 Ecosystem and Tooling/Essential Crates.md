---
tags: [rust, crates, ecosystem]
---

# Essential Crates

> [!summary] Summary
> Rust's standard library is deliberately minimal — much of what feels like "core" functionality in everyday Rust (async runtimes, serialization, error handling helpers) actually lives in a small set of extremely widely used **crates** from crates.io. This note surveys the ones a working Rust developer encounters constantly.

## 1. serde — serialization/deserialization

```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct User {
    name: String,
    age: u32,
}

let json = serde_json::to_string(&User { name: "Alice".into(), age: 30 })?;
let user: User = serde_json::from_str(&json)?;
```

`serde` itself defines the generic `Serialize`/`Deserialize` traits and derive macros (see [[Macros]] §3); format-specific crates (`serde_json`, `serde_yaml`, `bincode`, `toml`) plug into that shared trait interface. This split is why adding support for a new format to an existing `#[derive(Serialize)]` struct usually requires no code changes at all — just adding the new format crate as a dependency.

## 2. tokio — the dominant async runtime

Covered in depth in [[Async Await]] §4 — provides the executor, non-blocking I/O primitives, timers, and synchronization tools (`tokio::sync::Mutex`, channels) needed to actually run `async`/`await` code, since the standard library deliberately ships none of this itself.

## 3. anyhow and thiserror — error handling ergonomics

```rust
// thiserror — for LIBRARY code: define a precise, typed error enum
use thiserror::Error;

#[derive(Error, Debug)]
enum AppError {
    #[error("could not read file: {0}")]
    Io(#[from] std::io::Error),
    #[error("invalid config: {0}")]
    Config(String),
}
```

```rust
// anyhow — for APPLICATION code: one flexible catch-all error type
use anyhow::{Result, Context};

fn load_config() -> Result<Config> {
    let text = std::fs::read_to_string("config.toml")
        .context("failed to read config file")?;
    Ok(toml::from_str(&text)?)
}
```

- **`thiserror`**: generates `Display`/`Error` implementations for a hand-defined error enum via `#[derive(Error)]` — appropriate for **libraries**, where callers need to `match` on specific, precisely-typed error variants (see [[The Question Mark Operator]] §4 for the manual version of what `#[from]` automates).
- **`anyhow`**: provides a single `anyhow::Error` type that any `std::error::Error` can convert into via `?`, plus `.context()` for adding human-readable context as errors propagate — appropriate for **application** code (binaries), where the caller (a human reading logs) usually just needs a good error message, not to programmatically distinguish error variants.

## 4. clap — command-line argument parsing

```rust
use clap::Parser;

#[derive(Parser)]
struct Args {
    #[arg(short, long)]
    name: String,
    #[arg(short, long, default_value_t = 1)]
    count: u8,
}

fn main() {
    let args = Args::parse();
    for _ in 0..args.count {
        println!("Hello, {}!", args.name);
    }
}
```

A derive-macro-driven CLI parser — declaring a struct with `#[derive(Parser)]` and field attributes automatically generates argument parsing, `--help` text, and validation, without hand-writing an argument-parsing loop.

## 5. rayon — effortless data parallelism

```rust
use rayon::prelude::*;

let sum: i32 = (1..1_000_000).into_par_iter().sum();   // parallel, just by changing .iter() to .into_par_iter()
```

`rayon` provides parallel iterator adapters that mirror the standard `Iterator` trait's API (see [[Iterators and Closures]]) almost exactly — converting a CPU-bound sequential iterator chain into one that automatically splits work across a thread pool (via work-stealing) often requires changing only the single method call that starts the chain.

## 6. Other commonly encountered crates

| Crate | Purpose |
|---|---|
| `reqwest` | HTTP client, built on tokio |
| `axum` / `actix-web` | web application frameworks |
| `sqlx` / `diesel` | database access (async query macros / compile-time-checked ORM respectively) |
| `regex` | regular expressions (not in the standard library) |
| `rand` | random number generation (also not in the standard library) |
| `tracing` | structured, async-aware logging/instrumentation |
| `itertools` | additional iterator adapters beyond the standard library's set |

> [!note] Why so much lives outside the standard library
> Rust's standard library deliberately favors a small, extremely stable core, leaving fast-moving or opinionated functionality (async runtimes, web frameworks, regex engines) to the crates.io ecosystem instead — allowing these to iterate, compete, and evolve independently of Rust's own release cadence, at the cost of needing to actively choose and learn a handful of "everyone basically uses this" third-party crates as part of learning idiomatic Rust.

## See also
- [[Cargo and the Rust Toolchain]]
- [[Async Await]]
- [[Macros]]

#rust #crates #ecosystem
