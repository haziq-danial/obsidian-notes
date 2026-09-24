---
tags: [redis, use-cases, pubsub]
---

# Pub/Sub Messaging

> [!summary] Summary
> Redis's Publish/Subscribe messaging lets clients broadcast messages to channels, delivered to whoever happens to be subscribed at that moment. It's simple and fast, but fundamentally **fire-and-forget** — for durable, replayable messaging, see [[Streams]] instead.

![[pubsub-pattern.svg]]

## 1. Basic commands

| Command | Effect |
|---|---|
| `SUBSCRIBE channel [channel ...]` | subscribe to exact channel name(s) |
| `PSUBSCRIBE pattern [pattern ...]` | subscribe using a glob pattern (e.g., `news.*`) |
| `PUBLISH channel message` | send a message to all current subscribers of a channel |
| `UNSUBSCRIBE` / `PUNSUBSCRIBE` | stop listening |
| `PUBSUB CHANNELS [pattern]` | introspect active channels |
| `PUBSUB NUMSUB channel [channel ...]` | count subscribers per channel |

```
# Terminal 1
SUBSCRIBE news

# Terminal 2
PUBLISH news "breaking: redis 8 released"

# Terminal 1 receives:
1) "message"
2) "news"
3) "breaking: redis 8 released"
```

## 2. Fire-and-forget semantics

> [!warning] No persistence, no replay
> If no client is subscribed to a channel when `PUBLISH` happens, the message is simply **dropped** — there is no buffering, queue, or delivery guarantee. A client that subscribes a moment too late has already missed it permanently. This is the fundamental trade-off versus [[Streams]], which persist entries and support consumer groups with acknowledgment.

## 3. Sharded Pub/Sub (Redis 7+, Cluster mode)

Regular `PUBLISH` in [[Redis Cluster|Cluster mode]] broadcasts to *every* node in the cluster (since any client could be subscribed via any node) — fine at small scale, but a scalability bottleneck with many channels/messages. **Sharded Pub/Sub** (`SPUBLISH`/`SSUBSCRIBE`) instead routes a channel's messages only to the cluster node that owns that channel name's hash slot, keeping Pub/Sub traffic local to relevant shards rather than fanned out cluster-wide.

## 4. Common use cases

- **Real-time notifications**: pushing live updates to connected clients (chat messages, live scores, presence updates) — often paired with WebSocket servers that subscribe to Redis channels and relay messages to browser connections.
- **Cache invalidation broadcast**: when one service updates data, it publishes an invalidation message so other services' local caches can evict the stale entry (see [[Caching Patterns]] §4).
- **Cross-service event fan-out** where durability/replay isn't required — e.g., "a new order was placed" notifications to multiple interested services that are expected to be online.
- **Keyspace notifications**: Redis can publish Pub/Sub events for its *own* internal operations (`notify-keyspace-events` config) — e.g., publishing an event whenever a key expires, letting applications react to expirations in real time without polling.

## 5. When to use Streams instead

| Need | Use |
|---|---|
| Fire-and-forget, only care about currently-connected subscribers | Pub/Sub |
| Guaranteed delivery, replay, consumer groups, acknowledgment | [[Streams]] |
| Simple presence/notification signal | Pub/Sub |
| Durable event log / job queue with retry | [[Streams]] |

## See also
- [[Streams]]
- [[Caching Patterns]]
- [[RESP Protocol]] — RESP3 Push type used for Pub/Sub delivery

#redis #use-cases #pubsub
