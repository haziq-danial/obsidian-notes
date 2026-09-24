---
tags: [redis, glossary, reference]
---

# Redis Glossary

> [!summary] Summary
> Quick-reference definitions for terms used throughout the vault. Each term links to the note where it's covered in depth.

### A
- **ACL (Access Control List)** — Redis 6+'s user/permission system controlling which commands and keys a client can access. See [[Security and ACLs]].
- **AOF (Append-Only File)** — persistence mechanism logging every write command for replay on restart. See [[AOF Append-Only File]].
- **AOF rewrite** — compaction of the AOF into a minimal command/RDB-preamble form. See [[AOF Append-Only File]].

### C
- **Cache-aside** — caching pattern where the app checks cache first, falling back to the database on a miss. See [[Caching Patterns]].
- **Cache stampede** — many concurrent cache misses simultaneously overwhelming the backing database. See [[Caching Patterns]].
- **Cluster bus** — the internal TCP port cluster nodes use to gossip state with each other. See [[Redis Cluster]].
- **Consumer group** — a Streams feature enabling cooperative, acknowledged processing of a stream by multiple consumers. See [[Streams]].
- **CRC16** — the hash function used to map keys to hash slots in Redis Cluster. See [[Redis Cluster]].

### E
- **Eviction policy** — the rule determining which keys are removed once `maxmemory` is reached. See [[Eviction Policies]].
- **EXPIRE / TTL** — mechanism for automatically deleting a key after a set duration. See [[Keys and Expiry]].

### F
- **Fork (copy-on-write)** — the OS mechanism Redis uses to snapshot data in a child process without blocking the main thread. See [[RDB Snapshotting]].
- **Full resync** — the initial RDB-based bulk transfer a new replica performs when first connecting. See [[Replication]].

### H
- **Hash slot** — one of 16384 fixed partitions of the keyspace used by Redis Cluster for sharding. See [[Redis Cluster]].
- **Hash tag** — the `{...}` portion of a key used to force co-location of related keys in the same hash slot. See [[Redis Cluster]].
- **HyperLogLog** — a probabilistic structure for approximate distinct counting in ~12KB. See [[Bitmaps and HyperLogLog]].

### L
- **LFU (Least Frequently Used)** — an eviction/tracking strategy based on access frequency. See [[Eviction Policies]].
- **Listpack** — a compact, contiguous memory encoding used for small Lists/Hashes/Sets/Sorted Sets. See [[Memory Optimization]].
- **LRU (Least Recently Used)** — an eviction/tracking strategy based on recency of access. See [[Eviction Policies]].
- **Lua scripting** — server-side atomic scripting via `EVAL`/`EVALSHA`/Functions. See [[Lua Scripting]].

### M
- **maxmemory** — the configured memory limit that triggers eviction once reached. See [[Eviction Policies]].
- **MESI-style redirect (MOVED/ASK)** — cluster redirect replies telling a client which node actually owns a key's slot. See [[Redis Cluster]].
- **MULTI/EXEC** — Redis's transaction mechanism for atomic, isolated command batches. See [[Pipelining and Transactions]].

### O
- **ODOWN (Objectively Down)** — a Sentinel quorum's agreed conclusion that a primary is unreachable. See [[Redis Sentinel]].

### P
- **Partial resync** — a replica reconnecting and receiving only missing commands from the replication backlog instead of a full RDB transfer. See [[Replication]].
- **Pipelining** — sending multiple commands without waiting for each reply, reducing round trips. See [[Pipelining and Transactions]].
- **Pub/Sub** — fire-and-forget publish/subscribe messaging between clients. See [[Pub Sub Messaging]].

### Q
- **Quicklist** — the internal encoding of a List: a linked list of compact listpack nodes. See [[Lists]].

### R
- **RDB (Redis Database)** — point-in-time binary snapshot persistence. See [[RDB Snapshotting]].
- **Replica** — a read-only copy of a primary's dataset kept in sync via replication. See [[Replication]].
- **Replication backlog** — a bounded buffer of recent writes enabling partial resync after brief disconnects. See [[Replication]].
- **RESP (REdis Serialization Protocol)** — the wire protocol clients use to communicate with Redis. See [[RESP Protocol]].

### S
- **SDOWN (Subjectively Down)** — one Sentinel's individual, unconfirmed view that a primary is unreachable. See [[Redis Sentinel]].
- **Sentinel** — Redis's monitoring and automatic-failover system for primary-replica setups. See [[Redis Sentinel]].
- **Skip list** — the probabilistic ordered structure underlying Sorted Sets' range queries. See [[Sorted Sets]].
- **Sliding window (rate limiting)** — a rate-limiting approach that avoids fixed-window boundary bursts. See [[Rate Limiting]].
- **Stream** — an append-only, replayable log data type with consumer-group support. See [[Streams]].

### T
- **Token bucket** — a rate-limiting algorithm allowing controlled bursts up to a capacity. See [[Rate Limiting]].
- **Transaction** — see MULTI/EXEC above. See [[Pipelining and Transactions]].

### W
- **WATCH** — optimistic-locking primitive that aborts a transaction if a watched key changes. See [[Pipelining and Transactions]].
- **Write-through / write-behind** — caching patterns where writes go to cache and database together (through) or cache first with async DB flush (behind). See [[Caching Patterns]].

## See also
- [[Redis MOC]]

#glossary #reference
