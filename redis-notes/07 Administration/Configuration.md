---
tags: [redis, administration, configuration]
---

# Configuration

> [!summary] Summary
> Redis is configured via `redis.conf` at startup and, for most parameters, dynamically at runtime via `CONFIG SET` — no restart required for the vast majority of tuning changes.

## 1. Configuration sources and precedence

1. Compile-time defaults.
2. `redis.conf` file, read at startup (`redis-server /path/to/redis.conf`).
3. Command-line arguments passed at startup (override the config file).
4. Runtime `CONFIG SET` changes (override everything above, but are **not** persisted to `redis.conf` automatically).

```
CONFIG GET maxmemory
CONFIG SET maxmemory 4gb
CONFIG REWRITE   # persist current runtime config back into redis.conf
```

> [!tip] Always CONFIG REWRITE after a meaningful CONFIG SET
> A `CONFIG SET` without a follow-up `CONFIG REWRITE` is lost on the next restart, silently reverting to whatever `redis.conf` still says — a common source of "I fixed this already, why is it broken again after a restart?" incidents.

## 2. Key configuration areas

| Area | Key settings |
|---|---|
| Networking | `bind`, `port`, `protected-mode`, `timeout` |
| Memory | `maxmemory`, `maxmemory-policy` (see [[Eviction Policies]]) |
| Persistence | `save`, `appendonly`, `appendfsync` (see [[RDB Snapshotting]], [[AOF Append-Only File]]) |
| Replication | `replicaof`, `replica-read-only`, `repl-backlog-size` (see [[Replication]]) |
| Security | `requirepass`, `aclfile`, `rename-command` (see [[Security and ACLs]]) |
| Logging | `loglevel`, `logfile` |
| Encodings/thresholds | `hash-max-listpack-entries`, `zset-max-listpack-entries`, etc. (see [[Memory Optimization]]) |

## 3. Config parameters that require a restart

The vast majority of settings apply immediately via `CONFIG SET`, but a small set of structural options (e.g., changing `port` while running, or certain persistence directory layouts) still require a full restart. Check `CONFIG GET <param>` behavior and the official docs for any setting before assuming it's live-tunable — attempting `CONFIG SET` on a restart-only parameter returns an explicit error rather than silently no-op'ing.

## 4. Useful introspection commands

| Command | Purpose |
|---|---|
| `INFO [section]` | comprehensive server state — memory, persistence, replication, stats, clients |
| `CONFIG GET *` | dump all current configuration |
| `CLIENT LIST` | list all connected clients and their state |
| `COMMAND COUNT` / `COMMAND DOCS` | introspect available commands |
| `DEBUG JMAP` / `LATENCY HISTORY` | deeper diagnostic tooling (see [[Monitoring and Observability]]) |

## 5. Multiple config files / includes

`redis.conf` supports `include /path/to/other.conf`, letting you split shared defaults from per-environment overrides (e.g., a common base config plus an environment-specific file for `maxmemory` and `bind`), which is standard practice for managing fleets of Redis instances via configuration management tools.

## See also
- [[Security and ACLs]]
- [[Memory Optimization]]
- [[Monitoring and Observability]]

#redis #administration #configuration
