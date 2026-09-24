---
tags: [redis, use-cases, rate-limiting]
---

# Rate Limiting

> [!summary] Summary
> Rate limiting — capping how many requests a client can make in a given window — is a natural fit for Redis's atomic counters and sorted sets. This note covers the three standard algorithms and their Redis implementations.

## 1. Fixed window counter

Simplest approach: count requests in discrete time buckets (e.g., per-minute), reset each new window.

```
key = "ratelimit:{user_id}:{current_minute}"
count = INCR key
if count == 1:
    EXPIRE key 60
if count > LIMIT:
    reject request
```

Using `INCR` (atomic, see [[Strings]] §2) makes the read-check-increment race-free without extra locking.

> [!warning] Boundary burst problem
> A client can send the full limit right at the end of one window and again immediately at the start of the next, achieving up to 2x the intended rate in a short burst straddling the window boundary. Fixed windows trade this imprecision for simplicity.

## 2. Sliding window log (via Sorted Set)

Store a timestamp per request in a [[Sorted Sets|Sorted Set]], scored by the request time, and count only entries within the trailing window — eliminating the boundary burst problem entirely.

```
ZADD ratelimit:{user_id} now request_id
ZREMRANGEBYSCORE ratelimit:{user_id} -inf (now - window_seconds)
count = ZCARD ratelimit:{user_id}
EXPIRE ratelimit:{user_id} window_seconds
if count > LIMIT:
    reject request
```

**Pros**: perfectly accurate sliding window, no boundary effects.
**Cons**: memory proportional to request count within the window (one Sorted Set member per request), more expensive per-check than a single `INCR`.

## 3. Sliding window counter (approximation)

A middle ground: combine the current and previous fixed windows with a weighted average based on how far into the current window we are — approximates a true sliding window using just two counters instead of one entry per request.

$$ \text{estimated count} = \text{count}_{\text{current}} + \text{count}_{\text{previous}} \times (1 - \frac{\text{elapsed in current window}}{\text{window size}}) $$

This is the algorithm behind many production API gateway rate limiters — nearly as accurate as the full sliding log, at fixed (two-counter) memory cost regardless of request volume.

## 4. Token bucket (via Lua script)

Models a bucket that refills at a steady rate and is drained per request — naturally supports **bursts** up to the bucket capacity while enforcing a long-term average rate.

```lua
-- simplified token bucket, run via EVAL for atomicity
local tokens_key = KEYS[1]
local timestamp_key = KEYS[2]
local rate = tonumber(ARGV[1])          -- tokens added per second
local capacity = tonumber(ARGV[2])
local now = tonumber(ARGV[3])
local requested = tonumber(ARGV[4])

local last_tokens = tonumber(redis.call('GET', tokens_key)) or capacity
local last_refreshed = tonumber(redis.call('GET', timestamp_key)) or now
local delta = math.max(0, now - last_refreshed)
local filled = math.min(capacity, last_tokens + delta * rate)

local allowed = filled >= requested
if allowed then
    filled = filled - requested
end

redis.call('SET', tokens_key, filled)
redis.call('SET', timestamp_key, now)
return allowed
```

Running this as a [[Lua Scripting|Lua script]] is essential — the read-compute-write sequence must be atomic, or two concurrent requests could both read the same token count and both be incorrectly allowed.

## 5. Choosing an algorithm

| Algorithm | Accuracy | Memory cost | Allows bursts |
|---|---|---|---|
| Fixed window | Low (boundary bursts) | Minimal (one counter) | Only at window boundaries (unintentionally) |
| Sliding window log | Perfect | O(requests in window) | No |
| Sliding window counter | High (approximation) | Minimal (two counters) | No |
| Token bucket | High | Minimal (two values) | Yes, intentionally, up to bucket capacity |

## See also
- [[Strings]] — atomic INCR
- [[Sorted Sets]]
- [[Lua Scripting]]

#redis #use-cases #rate-limiting
