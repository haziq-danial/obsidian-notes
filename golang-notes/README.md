# Go — Obsidian Vault

Open this folder (`golang-notes/`) as a vault in [Obsidian](https://obsidian.md) (`File → Open folder as vault`).

Start at **[[Go MOC]]** in the `00 MOC` folder — it links every note in the vault and gives a suggested reading order.

## Structure

```
golang-notes/
├── 00 MOC/                              → Map of Content (start here)
├── 01 Fundamentals/                     → what Go is, toolchain, variables/types, control flow, functions
├── 02 Composite Types/                  → arrays/slices, maps, structs, pointers
├── 03 Concurrency/                      → goroutines, channels/select, sync package, concurrency patterns
├── 04 Interfaces and Generics/          → methods, interfaces, embedding, generics
├── 05 Error Handling/                   → errors as values, panic/recover, error wrapping
├── 06 Packages and Modules/             → packages/visibility, Go modules
├── 07 Standard Library and Tooling/     → testing, key stdlib packages, go tool commands
├── 08 Memory and Runtime/               → scheduler, garbage collector, escape analysis
├── 09 Glossary/                         → quick-reference glossary
└── attachments/                         → SVG diagrams embedded throughout the notes
```

## Features used

- **Wikilinks** (`[[Note Name]]`) connect every topic — use Graph View to see the whole map.
- **Callouts** (`> [!note]`, `> [!tip]`, `> [!warning]`, `> [!example]`) highlight key ideas, gotchas, and worked examples.
- **Mermaid diagrams** for flowcharts/sequence/state diagrams (rendered natively by Obsidian ≥ 0.15).
- **Hand-drawn SVG diagrams** in `attachments/` for the concepts that benefit most from a picture: the build pipeline, slice internals (ptr/len/cap and reallocation), the GMP scheduler model, channels and select, what an interface value actually is, defer/panic/recover control flow, escape analysis (stack vs heap), error-wrapping chains, and Go module dependency resolution.
- **Tags** (`#go`, `#concurrency`, `#error-handling`, …) for cross-cutting filtering in the search/tag pane.
