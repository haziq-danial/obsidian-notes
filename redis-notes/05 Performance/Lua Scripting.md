---
tags: [redis, performance, lua, scripting]
---

# Lua Scripting

> [!summary] Summary
> Redis embeds a Lua interpreter, letting clients send small scripts that execute **atomically** on the server, combining multiple commands and custom logic into one indivisible operation — the most powerful tool for correctness-sensitive multi-step operations, going beyond what [[Pipelining and Transactions|MULTI/EXEC]] alone can express.

## 1. Why run logic on the server at all

Any multi-step "read a value, compute something, write a value" sequence run from the client has a potential race window between the read and the write (another client's command can interleave). Running that same logic as a Lua script executes it as a **single atomic unit** on Redis's single-threaded event loop (see [[Redis Architecture and Event Loop]]) — no other command runs until the script finishes.

## 2. Basic usage

```
EVAL "return redis.call('SET', KEYS[1], ARGV[1])" 1 mykey myvalue
```

- `KEYS[]` and `ARGV[]` are passed explicitly rather than letting the script embed key names directly — this convention lets [[Redis Cluster]] statically determine which keys a script touches (routing it to the correct node, and rejecting scripts whose keys span multiple hash slots) without having to parse/execute the script body.
- `redis.call(...)` invokes a Redis command from within the script; `redis.pcall(...)` does the same but catches errors instead of aborting the script.

> [!example] Atomic conditional decrement
> ```lua
> -- Only decrement if the balance would stay non-negative
> local balance = tonumber(redis.call('GET', KEYS[1]))
> if balance >= tonumber(ARGV[1]) then
>     return redis.call('DECRBY', KEYS[1], ARGV[1])
> else
>     return -1  -- insufficient balance
> end
> ```
> This check-then-act sequence is race-free specifically because it runs as one atomic script — no other client's command can observe or modify `balance` in between the check and the decrement.

## 3. Caching scripts: EVALSHA

Sending a full script body on every call wastes bandwidth for scripts run frequently. Instead:
1. `SCRIPT LOAD "<script body>"` — Redis compiles it, caches it, and returns a SHA1 hash.
2. `EVALSHA <sha1> numkeys key [key ...] arg [arg ...]` — runs the cached script by hash, without resending the body.

Most client libraries handle this transparently: they call `EVALSHA` first, and only fall back to a full `EVAL` (re-caching it) if the server replies `NOSCRIPT` (e.g., after a `SCRIPT FLUSH` or restart).

## 4. Functions (Redis 7+)

**Functions** are the modern, more structured evolution of ad-hoc `EVAL` scripts: named, versioned Lua libraries registered once with `FUNCTION LOAD` and invoked by name via `FCALL`, rather than distributing raw script bodies (and their SHA1 hashes) throughout application code. They support proper library organization and are easier to manage operationally (list, inspect, delete via `FUNCTION LIST`/`FUNCTION DELETE`) than loose cached scripts.

## 5. Constraints and gotchas

> [!warning] Scripts block the event loop
> A slow or infinite-looping script blocks *every other client* for its entire duration — the same single-threaded trade-off as any other command (see [[Redis Architecture and Event Loop]]), but magnified because a script can pack arbitrarily much work into one atomic unit. `lua-time-limit` (legacy scripts) can interrupt long-running **read-only** scripts, but a script containing writes cannot be safely interrupted mid-way (doing so could leave the dataset in an inconsistent partial state), so writing scripts must be kept genuinely fast.

- Scripts must be **deterministic** with respect to what they write, since the script's *effects* (the individual write commands it issued) — not the Lua source itself — are what gets propagated to the [[AOF Append-Only File|AOF]] and to [[Replication|replicas]]. Using `redis.call('TIME')` or non-deterministic constructs for anything that affects a write is a correctness hazard for exactly this reason (modern Redis mitigates much of this automatically by propagating effects rather than script source, but writing scripts as if they need to be deterministic remains good practice).
- Scripts touching multiple keys in [[Redis Cluster|Cluster mode]] must have all those keys hash to the same slot, exactly like multi-key transactions.

## See also
- [[Pipelining and Transactions]]
- [[Redis Architecture and Event Loop]]
- [[Redis Cluster]]

#redis #performance #lua
