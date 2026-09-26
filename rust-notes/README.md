# Rust — Obsidian Vault

Open this folder (`rust-notes/`) as a vault in [Obsidian](https://obsidian.md) (`File → Open folder as vault`).

Start at **[[Rust MOC]]** in the `00 MOC` folder — it links every note in the vault and gives a suggested reading order.

## Structure

```
rust-notes/
├── 00 MOC/                       → Map of Content (start here)
├── 01 Fundamentals/              → what Rust is, cargo/rustc, variables, types, control flow
├── 02 Ownership and Borrowing/   → ownership, references/borrowing, slices, lifetimes
├── 03 Types and Structures/      → structs, enums/pattern matching, generics, traits
├── 04 Error Handling/            → Option/Result, panics, the ? operator
├── 05 Collections and Iterators/ → Vec/HashMap/String, iterators and closures
├── 06 Concurrency and Async/     → threads, message passing, shared state, async/await
├── 07 Smart Pointers/            → Box, Rc/Arc, RefCell/Cell, Deref/Drop
├── 08 Advanced Topics/           → trait objects, macros, unsafe Rust, modules/crates
├── 09 Ecosystem and Tooling/     → testing, essential crates (serde, tokio, clippy)
├── 10 Glossary/                  → quick-reference glossary
└── attachments/                  → SVG diagrams embedded throughout the notes
```

## Features used

- **Wikilinks** (`[[Note Name]]`) connect every topic — use Graph View to see the whole map.
- **Callouts** (`> [!note]`, `> [!tip]`, `> [!warning]`, `> [!example]`) highlight key ideas, gotchas, and worked examples.
- **Mermaid diagrams** for flowcharts/state/sequence diagrams (rendered natively by Obsidian ≥ 0.15).
- **Hand-drawn SVG diagrams** in `attachments/` for the concepts that benefit most from a picture: stack vs heap, move semantics, the borrow checker's rules, lifetimes, enum pattern matching, smart pointer comparison, message-passing vs shared-state concurrency, the async/await state machine, and the Cargo workflow.
- **Tags** (`#rust`, `#ownership`, `#concurrency`, …) for cross-cutting filtering in the search/tag pane.
