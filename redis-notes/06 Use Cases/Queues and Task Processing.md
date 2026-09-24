---
tags: [redis, use-cases, queues]
---

# Queues and Task Processing

> [!summary] Summary
> Redis supports task/job queues at two levels of sophistication: a simple [[Lists|List]]-based queue for straightforward producer-consumer workloads, and [[Streams]] with consumer groups when reliable delivery, retries, and multiple independent consumer groups are needed.

## 1. Simple queue with Lists

```
# Producer
LPUSH queue:emails '{"to":"a@x.com","subject":"Welcome"}'

# Worker (blocks until work arrives)
BRPOP queue:emails 0
```

This is the minimal viable job queue: `LPUSH` from any number of producers, `BRPOP` from any number of workers — Redis's atomic list operations ensure each queued job is delivered to exactly one worker even with many workers polling concurrently (see [[Lists]] §2).

## 2. Reliable queue pattern

> [!warning] The basic BRPOP pattern can lose jobs
> If a worker calls `BRPOP`, receives a job, then crashes before finishing it, the job is gone — it was already removed from the list. For anything where losing a job is unacceptable, use `LMOVE`/`BLMOVE` into a per-worker "processing" list instead:

```
BLMOVE queue:emails queue:processing:worker1 LEFT RIGHT 0
# ... worker processes the job ...
LREM queue:processing:worker1 1 <job>   # acknowledge completion by removing it
```

If `worker1` crashes mid-job, the job remains visible in `queue:processing:worker1` — a monitoring process can periodically scan processing lists for orphaned jobs (from workers that stopped heartbeating) and `LMOVE` them back onto the main queue for another worker to pick up.

## 3. Priority queues via Sorted Sets

When jobs need priority ordering rather than strict FIFO, a [[Sorted Sets|Sorted Set]] scored by priority (or by scheduled execution time, for delayed jobs) fits naturally:

```
ZADD queue:scheduled 1700003600 '{"job":"send_reminder","id":42}'   # score = run-at unix time

# Worker polls for due jobs:
ZRANGEBYSCORE queue:scheduled -inf <now> LIMIT 0 1
# then ZREM the job once claimed, and process it
```

This is the standard pattern for **delayed/scheduled jobs** — nothing Redis-specific is needed beyond a Sorted Set and a polling worker (or `BZPOPMIN`-based blocking variant for immediate-priority queues without a delay component).

## 4. Streams for durable, replayable, multi-consumer processing

For workloads needing built-in acknowledgment tracking, automatic retry of stalled jobs, and multiple independent consumer groups reading the same feed, [[Streams]] are the more capable choice over Lists — see that note's consumer-group section for the full mechanics (`XREADGROUP`, `XACK`, `XCLAIM`/`XAUTOCLAIM`, the Pending Entries List).

## 5. Choosing between the options

| Need | Best fit |
|---|---|
| Simple FIFO queue, occasional job loss on crash tolerable | [[Lists]] with `BRPOP` |
| FIFO queue, job loss on crash unacceptable | Lists with `LMOVE` reliable-queue pattern |
| Priority or delayed/scheduled jobs | [[Sorted Sets]] |
| Guaranteed delivery, retries, multiple independent consumer groups, replay | [[Streams]] |
| Purely ephemeral, no persistence needed, broadcast semantics | [[Pub Sub Messaging]] |

> [!note] Redis vs a dedicated message broker
> For simple, latency-sensitive job queues already living alongside other Redis-based data, using Redis directly avoids operating a separate broker (RabbitMQ, Kafka, SQS). For very high-throughput, long-retention, or complex-routing messaging needs, a dedicated broker remains the better-specialized tool — Redis's queueing features are a strong "good enough, and it's already here" option rather than a full broker replacement.

## See also
- [[Lists]]
- [[Streams]]
- [[Sorted Sets]]

#redis #use-cases #queues
