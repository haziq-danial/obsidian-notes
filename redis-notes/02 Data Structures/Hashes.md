---
tags: [redis, data-structures, hashes]
---

# Hashes

> [!summary] Summary
> A Redis Hash is a field-value map attached to a single key — effectively a mini object/record stored under one top-level key, ideal for representing structured entities like a user profile or a product record.

## 1. Basic operations

| Command | Effect |
|---|---|
| `HSET key field value [field value ...]` | set one or more fields |
| `HGET key field` | get one field |
| `HMGET key f1 f2 ...` | get multiple fields |
| `HGETALL key` | get all fields and values |
| `HDEL key field [field ...]` | delete field(s) |
| `HEXISTS key field` | check if a field exists |
| `HINCRBY key field n` | atomically increment a numeric field |
| `HKEYS key` / `HVALS key` | list all field names / all values |
| `HLEN key` | number of fields |
| `HRANDFIELD key [count]` | random field(s) |

## 2. Hash vs separate Strings for an object

> [!example] Modeling a user
> **Option A — one Hash:**
> ```
> HSET user:1000 name "Alice" age 30 email "a@x.com"
> ```
> **Option B — separate Strings:**
> ```
> SET user:1000:name "Alice"
> SET user:1000:age 30
> SET user:1000:email "a@x.com"
> ```
> Option A is almost always better: one key instead of three (less keyspace overhead, see [[Memory Optimization]]), fields can be fetched/updated together or individually, and a small hash's compact internal encoding is dramatically cheaper per field than three separate top-level String objects.

## 3. Internal encoding: listpack vs hashtable

| Encoding | Used when |
|---|---|
| `listpack` | small hashes (few fields, short values) — a single compact, contiguous memory blob |
| `hashtable` | once the hash grows beyond `hash-max-listpack-entries` / `hash-max-listpack-value` thresholds |

The `listpack` encoding is why "many small hashes" is a common [[Memory Optimization]] technique: representing millions of small objects as compact hashes uses dramatically less memory than millions of top-level String keys, because each hash field avoids the per-key overhead paid by the global keyspace hash table.

> [!tip] Field-bucketing large collections
> A common pattern for very large object counts (e.g., millions of users) is to bucket several objects' fields into one hash keyed by a hashed ID range (`user:bucket:42 → {1000_name: "Alice", 1000_age: 30, 1001_name: ...}`), keeping each underlying hash small enough to stay in the efficient `listpack` encoding while cutting the number of top-level keys by orders of magnitude.

## 4. Common use cases

- User profiles / product records / any structured entity
- Representing objects that need partial updates (change one field without touching the rest)
- Session data (see [[Session Store]])
- Counters grouped per entity (e.g., `HINCRBY stats:video:42 views 1`)

## See also
- [[Strings]]
- [[Memory Optimization]]
- [[Session Store]]

#redis #data-structures #hashes
