---
tags: [redis, persistence, rdb]
---

# RDB Snapshotting

> [!summary] Summary
> RDB (Redis Database) persistence produces a single compact binary snapshot of the entire dataset at a point in time. It's fast to load, ideal for backups, but can lose data written since the last snapshot.

![[rdb-aof-persistence.svg]]

## 1. How a snapshot is taken

1. Redis calls `fork()`, creating a child process that shares the parent's memory pages via **copy-on-write (COW)**.
2. The child process walks the entire dataset and writes it to a temporary RDB file.
3. Because of COW, the parent process can keep serving reads/writes normally — the OS only duplicates a memory page if the parent modifies it while the child is still reading the old version of that page.
4. Once complete, the child atomically renames the temp file to `dump.rdb` and exits.

This is why the main event loop (see [[Redis Architecture and Event Loop]]) is barely interrupted during a snapshot — the actual disk-writing work happens in a separate OS process, not on the single command-processing thread.

## 2. Triggering a snapshot

| Command / config | Effect |
|---|---|
| `SAVE` | synchronous snapshot — **blocks** the main thread until done (avoid in production) |
| `BGSAVE` | asynchronous snapshot via fork — the normal way to snapshot |
| `save 900 1` / `save 300 10` / `save 60 10000` (config) | automatic `BGSAVE` if N changes occur within M seconds — multiple rules can be combined |
| `save ""` | disable automatic snapshotting entirely |

Redis also always performs a final `SAVE` on graceful shutdown (unless disabled), and a replica performs an RDB-based full resync when it first connects to a primary (see [[Replication]]).

## 3. Trade-offs

> [!warning] Data-loss window
> If Redis crashes between snapshots, **all writes since the last successful snapshot are lost**. A `save 300 10` rule means up to 5 minutes of writes (if fewer than 10 keys changed) could vanish on a crash — this is the central trade-off RDB makes for its speed and compactness.

**Advantages:**
- Extremely compact single file — ideal for backups, disaster recovery, and quickly seeding new replicas.
- Fast to load on restart — a raw binary dump, no command replay needed.
- Minimal runtime overhead during normal operation (the fork/COW cost is proportional to how much memory changes during the snapshot, not dataset size).

**Disadvantages:**
- Coarse durability — data loss is measured in the interval between snapshots, not seconds.
- `fork()` on a very large dataset (tens of GB) can itself take a non-trivial amount of time and memory (COW can double memory usage under a heavy write load during the snapshot) — a capacity-planning consideration.

## 4. RDB file format essentials

The RDB file is a compact, versioned binary format: a header, then a series of key-value entries (each tagged with its type and, if applicable, an expiry timestamp), ending with a checksum. `redis-check-rdb` validates a file's integrity, useful when investigating a corrupted or truncated snapshot.

## See also
- [[AOF Append-Only File]]
- [[Backup and Restore]]
- [[Replication]] — full resync uses an RDB transfer

#redis #persistence #rdb
