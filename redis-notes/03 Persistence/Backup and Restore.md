---
tags: [redis, persistence, backup]
---

# Backup and Restore

> [!summary] Summary
> Practical procedures for backing up and restoring a Redis instance, building on [[RDB Snapshotting]] and [[AOF Append-Only File]].

## 1. Backing up via RDB

The simplest, most portable backup is just the RDB file itself:

```bash
redis-cli BGSAVE
# wait for completion (check `rdb_bgsave_in_progress` via INFO persistence)
cp /var/lib/redis/dump.rdb /backups/dump-$(date +%F).rdb
```

> [!warning] Don't copy dump.rdb while a BGSAVE is in progress
> The file is being written by the child process; copying mid-write yields a truncated/corrupt snapshot. Poll `INFO persistence` for `rdb_bgsave_in_progress: 0` before copying, or better, use `--rdb` on `redis-cli` which handles this correctly.

For a live, non-disruptive backup without triggering a new fork on a busy primary, it's common to run `BGSAVE`/backup against a [[Replication|replica]] instead of the primary.

## 2. Backing up via AOF

Back up the entire `appendonlydir/` directory (manifest + base + incremental files, see [[AOF Append-Only File]] §4), keeping all parts consistent as a set — copying only one part leaves an unusable backup.

## 3. Restoring

1. Stop the Redis server (or point it at a fresh data directory).
2. Place the backed-up `dump.rdb` (and/or `appendonlydir/`) into the configured `dir`.
3. Start Redis — it automatically loads AOF if enabled (`appendonly yes`), otherwise falls back to RDB.

```bash
systemctl stop redis
cp /backups/dump-2024-01-01.rdb /var/lib/redis/dump.rdb
systemctl start redis
redis-cli DBSIZE   # sanity-check the restore
```

## 4. Validating backup integrity

| Tool | Purpose |
|---|---|
| `redis-check-rdb <file>` | validates an RDB file's structure without loading it into a live server |
| `redis-check-aof <file>` | validates (and can truncate a corrupted tail from) an AOF file |

> [!tip] Test restores, not just backups
> A backup you've never restored is unverified. Periodically restore a backup into a scratch instance and sanity-check key counts (`DBSIZE`), spot-check known keys, and confirm the application can read from it — this catches silent corruption or incomplete backup scripts before an actual incident.

## 5. Point-in-time considerations

Neither RDB nor AOF alone gives arbitrary point-in-time recovery the way a WAL-based relational database might:
- RDB only has the discrete points when a snapshot was taken.
- AOF *can* be truncated to an earlier point (since it's a sequential command log) to approximate replaying up to just before a bad command — e.g., recovering from an accidental `FLUSHALL` by truncating the AOF to exclude that command, though this requires careful manual surgery on the file and is not push-button.

For applications needing true point-in-time recovery guarantees, Redis is typically paired with a system-level snapshotting approach (filesystem/volume snapshots) taken frequently alongside AOF.

## See also
- [[RDB Snapshotting]]
- [[AOF Append-Only File]]
- [[Configuration]]

#redis #persistence #backup
