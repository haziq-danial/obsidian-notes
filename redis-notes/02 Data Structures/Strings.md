---
tags: [redis, data-structures, strings]
---

# Strings

> [!summary] Summary
> The String is Redis's simplest and most versatile type: a binary-safe byte sequence up to 512MB. It's used not just for text but for counters, serialized objects, binary blobs, and bitmaps.

![[redis-data-types.svg]]

## 1. Basic operations

| Command | Effect |
|---|---|
| `SET key value` | set a string value |
| `GET key` | retrieve a string value |
| `SET key value EX seconds` | set with expiry, see [[Keys and Expiry]] |
| `MSET k1 v1 k2 v2 ...` | set multiple keys atomically |
| `MGET k1 k2 ...` | get multiple keys atomically |
| `APPEND key value` | append to an existing string (or create it) |
| `STRLEN key` | length in bytes |
| `GETRANGE key start end` | substring by byte offsets |
| `SETRANGE key offset value` | overwrite part of a string at a byte offset |

## 2. Atomic counters

Because Redis executes commands atomically on its single thread (see [[Redis Architecture and Event Loop]]), numeric strings support atomic increment/decrement — the classic building block for counters, rate limiters, and IDs, entirely without a client-side read-modify-write race.

| Command | Effect |
|---|---|
| `INCR key` | increment by 1 (creates key at 0 first if missing) |
| `INCRBY key n` | increment by n |
| `DECR key` / `DECRBY key n` | decrement |
| `INCRBYFLOAT key f` | increment by a floating-point amount |

> [!example] Why atomicity matters here
> Without server-side atomic increment, a naive `GET` → add 1 in the client → `SET` sequence has a race: two clients reading the same value simultaneously both compute `old + 1` and one increment is lost. `INCR` performs the read-modify-write entirely inside the single-threaded server, making the race impossible.

## 3. Strings as binary data / bitmaps

Since strings are binary-safe, they double as the storage for [[Bitmaps and HyperLogLog|bitmap operations]] (`SETBIT`, `GETBIT`, `BITCOUNT`, `BITOP`) — a string is just a byte array, and bit-level commands index directly into those bytes.

## 4. Internal encodings

Redis chooses the cheapest internal representation automatically, transparent to the client:

| Encoding | Used when |
|---|---|
| `int` | value is a number that fits in a long |
| `embstr` | short strings (≤ 44 bytes) stored contiguously with their object header — one allocation, cache-friendly |
| `raw` | longer strings — a separate allocation for the buffer |

This matters for [[Memory Optimization]]: short numeric or short-string keys are meaningfully cheaper than long ones due to allocator overhead per object, not just the raw byte count.

## 5. Common use cases

- Caching serialized objects (JSON, MessagePack, Protocol Buffers) — see [[Caching Patterns]]
- Atomic counters (page views, like counts, rate-limit windows — see [[Rate Limiting]])
- Distributed locks (`SET key val NX EX ttl`, see [[Keys and Expiry]] §4)
- Feature flags / simple config values
- Session tokens (see [[Session Store]])

## See also
- [[Keys and Expiry]]
- [[Bitmaps and HyperLogLog]]
- [[Memory Optimization]]

#redis #data-structures #strings
