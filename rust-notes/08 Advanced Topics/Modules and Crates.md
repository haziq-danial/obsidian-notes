---
tags: [rust, modules, crates]
---

# Modules and Crates

> [!summary] Summary
> Rust organizes code into a hierarchy: **crates** (compilation units, either a binary or a library) contain **modules** (namespaces within a crate), giving control over both code organization and, critically, **visibility** — what's exposed publicly versus kept as an implementation detail.

## 1. Crates — the unit of compilation

A **crate** is the smallest unit `rustc` compiles at a time — either a **binary crate** (has a `main` function, produces an executable) or a **library crate** (no `main`, produces a `.rlib` for other crates to depend on). A Cargo **package** can contain multiple crates (conventionally one library crate plus one or more binary crates sharing that library) — see [[Cargo and the Rust Toolchain]] for the project-layout side of this.

## 2. Modules — organizing code within a crate

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() { /* ... */ }
    }

    mod serving {                          // private by default — not `pub`
        fn take_order() { /* ... */ }
    }
}

pub fn eat_at_restaurant() {
    front_of_house::hosting::add_to_waitlist();   // full path from the crate root
}
```

- Everything is **private by default** — a module, function, struct, or field is invisible outside its defining module unless explicitly marked `pub`.
- A child module can always access its ancestors' private items; the reverse requires `pub`.

## 3. The privacy design philosophy

> [!note] Private by default is a deliberate API-design forcing function
> Because nothing is accessible outside its module without an explicit `pub`, the author of a module must consciously decide what's part of its public contract versus its private implementation details — every `pub` is a promise to callers that shouldn't be casually broken later. This mirrors the "immutable by default" philosophy from [[Variables and Mutability]]: Rust consistently defaults to the more restrictive, safer option, requiring an explicit opt-in for anything looser.

## 4. use — bringing paths into scope

```rust
use std::collections::HashMap;
use crate::front_of_house::hosting;   // absolute path from the crate root

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();   // no longer need the full path every time
}
```

```rust
pub use crate::front_of_house::hosting;   // re-export: makes `hosting` part of THIS module's public API too
```

`pub use` (re-exporting) is commonly used to present a clean, curated public API at a crate's root, even when the actual implementation is organized across many internal, deeper modules — callers see a flat, convenient structure while internal organization stays however granular makes sense for maintainers.

## 5. Splitting modules across files

```rust
// src/lib.rs
mod front_of_house;   // looks for src/front_of_house.rs OR src/front_of_house/mod.rs
```

```rust
// src/front_of_house.rs
pub mod hosting;      // looks for src/front_of_house/hosting.rs
```

As a crate grows, modules are typically split one-per-file (or one-per-directory for a module with its own submodules), while the `mod` declarations themselves stay in the parent, purely establishing the module tree's shape — the file layout mirrors the module hierarchy.

## 6. Visibility beyond pub

| Visibility | Meaning |
|---|---|
| (default, no keyword) | visible only within the current module and its descendants |
| `pub` | visible to anything that can reach this module at all |
| `pub(crate)` | visible anywhere within the current crate, but not to external dependents |
| `pub(super)` | visible only to the parent module |
| `pub(in path)` | visible only within a specific, explicitly named module subtree |

`pub(crate)` is especially common for items that need to be shared across a crate's own internal modules but should never be considered part of the crate's external API surface for downstream users.

## See also
- [[Cargo and the Rust Toolchain]]
- [[Traits]]
- [[Unsafe Rust]]

#rust #modules #crates
