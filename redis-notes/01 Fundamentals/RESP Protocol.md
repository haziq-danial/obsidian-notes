---
tags: [redis, protocol, resp]
---

# RESP Protocol

> [!summary] Summary
> RESP (REdis Serialization Protocol) is the simple, text-based wire protocol clients use to talk to Redis. Its simplicity — easy to parse, human-readable, fast — is part of why writing a Redis client library is approachable and why `redis-cli` / raw `telnet` sessions work at all.

## 1. Why a custom protocol

RESP was designed for three goals: **simple to implement**, **fast to parse**, and **human-readable enough to debug by eye**. Unlike binary protocols requiring a schema/IDL, a RESP client can be written in a few dozen lines.

## 2. RESP2 data types

Each reply begins with a type-indicating byte:

| Prefix | Type | Example |
|---|---|---|
| `+` | Simple String | `+OK\r\n` |
| `-` | Error | `-ERR unknown command\r\n` |
| `:` | Integer | `:1000\r\n` |
| `$` | Bulk String | `$5\r\nhello\r\n` (length-prefixed, binary-safe) |
| `*` | Array | `*2\r\n$3\r\nfoo\r\n$3\r\nbar\r\n` (an array of 2 bulk strings) |
| `$-1` | Null bulk string | `$-1\r\n` |
| `*-1` | Null array | `*-1\r\n` |

> [!example] A raw client command and reply
> Client sends: `*3\r\n$3\r\nSET\r\n$3\r\nfoo\r\n$3\r\nbar\r\n`
> (An array of 3 bulk strings: `SET`, `foo`, `bar` — this is literally how every command, including `SET foo bar`, is encoded on the wire.)
>
> Server replies: `+OK\r\n`

Every client **command** is sent as a RESP array of bulk strings — there's no separate "command syntax," commands and their arguments use the exact same array-of-bulk-strings encoding as any other array reply.

## 3. RESP3 (Redis 6+)

RESP3 is an opt-in protocol upgrade (negotiated via the `HELLO` command) adding richer types while remaining backward compatible:

| New type | Purpose |
|---|---|
| Map (`%`) | Key-value replies (e.g., `HGETALL`) as a proper map instead of a flat array |
| Set (`~`) | Distinguish set replies from arrays |
| Double (`,`) | Native floating-point replies instead of bulk-string-encoded numbers |
| Boolean (`#`) | True/false replies |
| Big Number (`(`) | Arbitrary-precision integers |
| Push (`>`) | Out-of-band messages (used for Pub/Sub and client-side caching invalidation pushes) |
| Verbatim String | A string with a known format hint (e.g., markdown) |
| Null (`_`) | A single unified null type (replacing separate null bulk-string/null-array) |

RESP3's **Push type** is what enables **client-side caching** (tracking): the server can push invalidation messages to a client asynchronously, outside the normal request/reply flow, letting clients maintain a local cache of recently-read keys that's invalidated the instant the server-side value changes.

## 4. Inline commands

For very simple debugging (e.g., over raw `telnet`/`nc`), Redis also accepts an even simpler **inline command** format: a plain line of space-separated text terminated by `\r\n`, without the full RESP array encoding — useful only for manual testing, never used by real client libraries.

## 5. Why this matters in practice

- Understanding RESP explains why Redis commands are so uniform: every command is fundamentally "array of strings in, typed reply out," which is why pipelining (see [[Pipelining and Transactions]]) is trivial — just concatenate multiple RESP-encoded commands and read the replies back in order, no special batching protocol needed.
- It explains why writing a minimal Redis client is a common "learn a language" exercise — the protocol genuinely is that simple.

## See also
- [[Redis Architecture and Event Loop]]
- [[Pipelining and Transactions]]
- [[Pub Sub Messaging]] — RESP3 Push type

#redis #protocol
