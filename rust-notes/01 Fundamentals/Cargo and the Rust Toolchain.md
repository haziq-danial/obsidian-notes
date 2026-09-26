---
tags: [rust, cargo, toolchain]
---

# Cargo and the Rust Toolchain

> [!summary] Summary
> Cargo is Rust's official build tool, package manager, and project scaffolding system all in one — nearly every Rust project is built, tested, and distributed through it. `rustup` manages the underlying compiler toolchain itself.

![[cargo-workflow.svg]]

## 1. rustup: managing the compiler

`rustup` installs and manages Rust toolchains (compiler + standard library + Cargo), including switching between **stable**, **beta**, and **nightly** release channels, and between target platforms for cross-compilation.

```bash
rustup install stable
rustup default stable
rustup update
rustup target add wasm32-unknown-unknown   # add a cross-compilation target
```

- **Stable**: the standard channel for production code — new features only land here once fully stabilized.
- **Beta**: a preview of the next stable release, used for testing upcoming changes.
- **Nightly**: bleeding-edge, includes unstable/experimental features gated behind feature flags — required for some advanced/unstable APIs, common in library development that needs to test against upcoming compiler behavior.

## 2. Project structure

```bash
cargo new my_project      # binary project (has a main.rs)
cargo new my_lib --lib    # library project (has a lib.rs)
```

```
my_project/
├── Cargo.toml     ← manifest: metadata, dependencies, build profile
├── Cargo.lock     ← exact resolved dependency versions (commit this for binaries)
└── src/
    └── main.rs    ← entry point (or lib.rs for a library)
```

### Cargo.toml essentials

```toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2021"

[dependencies]
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1", features = ["full"] }

[dev-dependencies]
proptest = "1.0"   # only needed for tests, not the final build
```

The **edition** (2015, 2018, 2021, 2024) selects a set of language behaviors/syntax the crate opts into — editions let the language evolve (even in backward-incompatible ways) without breaking existing code, since each crate declares which edition's rules it was written against, and the compiler supports all editions simultaneously, even mixed across a dependency graph.

## 3. Core Cargo commands

| Command | Effect |
|---|---|
| `cargo build` | compile the project (debug profile: fast compile, unoptimized) |
| `cargo build --release` | compile with optimizations (slower compile, faster binary) |
| `cargo run` | build (if needed) and run the resulting binary |
| `cargo check` | type-check without producing a binary — much faster feedback loop while developing |
| `cargo test` | compile and run tests (see [[Testing in Rust]]) |
| `cargo doc --open` | generate and open HTML documentation from doc comments |
| `cargo clippy` | run Rust's linter for common mistakes and non-idiomatic patterns |
| `cargo fmt` | auto-format code to the standard style |
| `cargo add <crate>` | add a dependency to Cargo.toml |
| `cargo update` | update dependencies within their allowed semver ranges, updating Cargo.lock |
| `cargo publish` | publish a crate to crates.io |

> [!tip] cargo check as the default inner-loop command
> `cargo check` skips code generation entirely, only running the type checker and borrow checker — often several times faster than a full `cargo build`. Most Rust developers run `cargo check` constantly while writing code and reserve `cargo build`/`cargo run` for when they actually need to execute the result.

## 4. Cargo.lock and reproducible builds

`Cargo.toml` specifies version *ranges* (semver-compatible), while `Cargo.lock` pins the *exact* versions actually resolved and used for a build. For a **binary** application, committing `Cargo.lock` ensures every build (CI, teammates, production) uses identical dependency versions. For a **library** crate, `Cargo.lock` is typically not committed, since downstream consumers will resolve their own compatible versions as part of their own build.

## 5. Workspaces

A **workspace** groups multiple related crates (e.g., a binary plus several internal libraries) under one top-level `Cargo.toml`, sharing a single `Cargo.lock` and `target/` build directory — avoiding redundant compilation and version drift across crates that are developed together.

```toml
[workspace]
members = ["app", "core-lib", "cli-tool"]
```

## See also
- [[Essential Crates]]
- [[Testing in Rust]]
- [[Modules and Crates]]

#rust #cargo #toolchain
