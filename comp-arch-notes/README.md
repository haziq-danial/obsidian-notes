# Computer Architecture — Obsidian Vault

Open this folder (`comp-arch-notes/`) as a vault in [Obsidian](https://obsidian.md) (`File → Open folder as vault`).

Start at **[[Computer Architecture MOC]]** in the `00 MOC` folder — it links every note in the vault and gives a suggested reading order.

## Structure

```
comp-arch-notes/
├── 00 MOC/                 → Map of Content (start here)
├── 01 Fundamentals/        → number systems, boolean algebra, logic circuits
├── 02 CPU Architecture/    → ISA, datapath, pipelining, superscalar
├── 03 Memory/              → memory hierarchy, caches, virtual memory
├── 04 IO and Buses/        → buses, interrupts, DMA
├── 05 Parallelism/         → Flynn's taxonomy, multicore, GPUs
├── 06 Performance/         → metrics, Amdahl's Law
├── 07 Glossary/            → quick-reference glossary
└── attachments/            → SVG diagrams embedded throughout the notes
```

## Features used

- **Wikilinks** (`[[Note Name]]`) connect every topic — use Graph View to see the whole map.
- **Callouts** (`> [!note]`, `> [!tip]`, `> [!warning]`, `> [!example]`) highlight key ideas, gotchas, and worked examples.
- **Mermaid diagrams** for flowcharts/state machines (rendered natively by Obsidian ≥ 0.15).
- **Hand-drawn SVG diagrams** in `attachments/` for circuit- and architecture-level illustrations (Von Neumann vs Harvard, logic gates, datapath, pipeline timing, memory hierarchy, cache mapping, bus architecture, Flynn's taxonomy).
- **Tags** (`#cpu`, `#memory`, `#parallelism`, …) for cross-cutting filtering in the search/tag pane.
