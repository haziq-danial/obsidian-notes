# Redis — Obsidian Vault

Open this folder (`redis-notes/`) as a vault in [Obsidian](https://obsidian.md) (`File → Open folder as vault`).

Start at **[[Redis MOC]]** in the `00 MOC` folder — it links every note in the vault and gives a suggested reading order.

## Structure

```
redis-notes/
├── 00 MOC/                     → Map of Content (start here)
├── 01 Fundamentals/            → what Redis is, event loop, RESP protocol, keys/TTL
├── 02 Data Structures/         → strings, lists, hashes, sets, zsets, streams, bitmaps/HLL, geo
├── 03 Persistence/             → RDB, AOF, backup/restore
├── 04 Replication and HA/      → replication, Sentinel, Cluster
├── 05 Performance/             → memory optimization, eviction, pipelining/transactions, Lua
├── 06 Use Cases/               → caching, pub/sub, rate limiting, leaderboards, sessions, queues
├── 07 Administration/          → configuration, security/ACLs, monitoring
├── 08 Glossary/                → quick-reference glossary
└── attachments/                → SVG diagrams embedded throughout the notes
```

## Features used

- **Wikilinks** (`[[Note Name]]`) connect every topic — use Graph View to see the whole map.
- **Callouts** (`> [!note]`, `> [!tip]`, `> [!warning]`, `> [!example]`) highlight key ideas, gotchas, and worked examples.
- **Mermaid diagrams** for flowcharts/sequence/state diagrams (rendered natively by Obsidian ≥ 0.15).
- **Hand-drawn SVG diagrams** in `attachments/` for architecture- and system-level illustrations (event loop, data type internals, RDB/AOF persistence, replication topology, Sentinel failover, cluster hash-slot sharding, pub/sub, pipelining vs transactions, eviction policies).
- **Tags** (`#redis`, `#data-structures`, `#replication`, …) for cross-cutting filtering in the search/tag pane.
