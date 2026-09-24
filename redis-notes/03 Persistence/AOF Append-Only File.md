---
tags: [redis, persistence, aof]
---

# AOF (Append-Only File)

> [!summary] Summary
> AOF persistence logs every write command as it happens, appended to a file. Replaying that log reconstructs the dataset — trading larger files and slightly more overhead for a much smaller data-loss window than [[RDB Snapshotting]].

![[rdb-aof-persistence.svg]]

## 1. How it works

Every write command that modifies the dataset is appended to the AOF, encoded in [[RESP Protocol|RESP]] format — literally the same wire format used for client communication, which is part of why AOF files are simple to inspect and even hand-edit if truly necessary.

```
*3\r\n$3\r\nSET\r\n$3\r\nfoo\r\n$3\r\nbar\r\n
*3\r\n$4\r\nHSET\r\n$4\r\nuser\r\n$4\r\nname\r\n...
*2\r\n$4\r\nINCR\r\n$3\r\nctr\r\n
```

On restart, Redis simply **replays every command in order** against an empty dataset to reconstruct the exact final state.

## 2. fsync policy — the real durability knob

Writing to the AOF file buffer doesn't guarantee it's durably on disk — that requires an `fsync()` syscall, which is comparatively slow. The `appendfsync` setting controls the trade-off:

| Policy | Behavior | Durability | Performance |
|---|---|---|---|
| `always` | fsync after every single write command | Best — practically zero data loss | Slowest — every write pays disk-sync latency |
| `everysec` (default) | fsync once per second in a background thread | Good — up to ~1 second of writes at risk on a crash | Fast — the common production choice |
| `no` | let the OS decide when to flush | Weakest — depends entirely on OS buffer flush timing (can be tens of seconds) | Fastest |

> [!tip] Why `everysec` is the default
> It captures the vast majority of the durability benefit (bounding loss to roughly one second of writes, versus RDB's minutes) while keeping the fsync cost off the hot path of every individual command — a background thread handles the periodic fsync instead.

## 3. AOF rewriting (compaction)

Since AOF logs every command forever, it would grow unboundedly (e.g., a million `INCR` calls on the same counter logs a million lines for what is, in the end, just one final integer value). **AOF rewrite** compacts this:

1. Redis forks a child process (same COW mechanism as [[RDB Snapshotting|RDB]]).
2. The child writes the **current dataset state** in the most compact command form possible (or, since Redis 7, an RDB-format preamble followed by any writes that occurred during the rewrite).
3. The new, compact AOF replaces the old one atomically.

| Command / config | Effect |
|---|---|
| `BGREWRITEAOF` | manually trigger a rewrite |
| `auto-aof-rewrite-percentage 100` | auto-rewrite once the AOF has grown 100% since the last rewrite |
| `auto-aof-rewrite-min-size 64mb` | don't bother auto-rewriting below this size |

## 4. Multi-Part AOF (Redis 7+)

Modern Redis splits the AOF into a **manifest** file referencing a **base file** (the RDB-format snapshot from the last rewrite) plus one or more **incremental files** (commands since that rewrite) — avoiding the need to rewrite the entire history into one file from scratch each time, and making the directory structure (`appendonlydir/`) easier to reason about and back up consistently.

## 5. RDB vs AOF — practical guidance

| | RDB | AOF |
|---|---|---|
| Data-loss window | Minutes (per save rule) | ~1s (`everysec`) or 0 (`always`) |
| File size | Compact | Larger (though rewrite compacts it) |
| Restart speed | Fast (direct binary load) | Slower (must replay/parse commands, though the RDB-preamble base file helps) |
| Best for | Backups, fast full restores, seeding replicas | Minimizing data loss on crash |

> [!note] The common production setup
> Enable **both**: RDB for fast, compact backups and quick disaster recovery, AOF (`everysec`) to bound data loss between snapshots. On restart with both enabled, Redis loads from AOF (the more complete, more recent source of truth) rather than the RDB file.

## See also
- [[RDB Snapshotting]]
- [[Backup and Restore]]
- [[Redis Architecture and Event Loop]] — fork-based background rewrite

#redis #persistence #aof
