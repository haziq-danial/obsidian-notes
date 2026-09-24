---
tags: [redis, administration, monitoring]
---

# Monitoring and Observability

> [!summary] Summary
> Redis exposes rich runtime introspection via `INFO`, slow-query logging, latency diagnostics, and client-connection visibility — the tools for understanding what a running instance is actually doing and diagnosing performance problems.

## 1. INFO — the primary health snapshot

```
INFO
INFO memory
INFO replication
INFO stats
INFO clients
```

Key fields to watch:

| Field | Section | Meaning |
|---|---|---|
| `used_memory`, `used_memory_rss` | memory | actual vs OS-reported memory; ratio indicates fragmentation (see [[Memory Optimization]]) |
| `connected_clients` | clients | current connection count |
| `blocked_clients` | clients | clients waiting on `BLPOP`/`BRPOP`/etc. |
| `instantaneous_ops_per_sec` | stats | current throughput |
| `keyspace_hits` / `keyspace_misses` | stats | cache hit ratio — critical for [[Caching Patterns|caching]] workloads |
| `rdb_last_save_time`, `rdb_changes_since_last_save` | persistence | how current the last snapshot is |
| `master_repl_offset`, `slave_repl_offset` | replication | replication lag indicators (see [[Replication]]) |
| `role`, `connected_slaves` | replication | current role and replica count |

## 2. Slow log

Logs any command that exceeds a configurable execution-time threshold — the primary tool for finding what's actually causing latency on a single-threaded server (see [[Redis Architecture and Event Loop]]).

```
CONFIG SET slowlog-log-slower-than 10000   # microseconds (10ms)
SLOWLOG GET 10                              # last 10 slow entries
SLOWLOG RESET
```

> [!tip] Slow log is a first stop for latency investigations
> Since a single slow command blocks every other client on the same thread, an unexplained latency spike across *many* unrelated clients is a strong signal to check the slow log for one specific offending command (often an accidental `KEYS *`, an unexpectedly large `SORT`, or a Lua script with an unbounded loop) rather than assuming a general capacity problem.

## 3. Latency monitoring

A more targeted tool than the slow log for diagnosing specific classes of internal delay (fork time, command execution, expire cycles, AOF fsync stalls):

```
CONFIG SET latency-monitor-threshold 100   # ms
LATENCY HISTORY <event-name>
LATENCY LATEST
LATENCY DOCTOR                              # human-readable analysis
```

## 4. Client connection introspection

```
CLIENT LIST
CLIENT INFO
CLIENT KILL ID <id>
CLIENT NO-EVICT on
```

`CLIENT LIST` shows every connection's address, age, idle time, last command, and buffer sizes — useful for spotting a misbehaving client (e.g., one accumulating a huge output buffer because it's not reading replies fast enough).

## 5. MONITOR (debugging only — never in production)

```
MONITOR
```

Streams **every command** processed by the server in real time — extremely useful for interactive debugging on a quiet development instance, but imposes significant overhead and should never be left running against a production instance under real load.

## 6. Metrics for external monitoring systems

Most production deployments scrape `INFO` output (or use the Redis Exporter for Prometheus, or a managed cloud provider's built-in metrics) on an interval, tracking over time:
- Memory usage vs `maxmemory` (approaching the limit signals a need for [[Eviction Policies|eviction policy]] review or scaling)
- Hit/miss ratio trend (a dropping hit ratio often signals a cache-sizing or TTL problem)
- Replication lag (`master_repl_offset` delta between primary and replicas)
- Connected/blocked client counts (a rising blocked-client count can indicate a queue backing up)
- Command latency percentiles, ideally per command type

## See also
- [[Configuration]]
- [[Memory Optimization]]
- [[Redis Architecture and Event Loop]]

#redis #administration #monitoring
