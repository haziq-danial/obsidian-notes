---
tags: [redis, use-cases, leaderboards]
---

# Leaderboards

> [!summary] Summary
> Real-time ranked leaderboards are one of the cleanest showcases of [[Sorted Sets]]: a single data structure gives O(log n) score updates, O(log n) rank lookups, and O(log n + m) range queries for "top N" or "players around me."

## 1. The core model

```
ZADD leaderboard:global 1500 "alice"
ZADD leaderboard:global 1200 "bob"
ZADD leaderboard:global 1800 "carol"
```

One [[Sorted Sets|Sorted Set]] per leaderboard (e.g., per game mode, per season), member = player ID, score = their current points/rating.

## 2. Common queries

| Need | Command |
|---|---|
| Update a player's score | `ZADD leaderboard score player` (overwrites) or `ZINCRBY leaderboard delta player` (relative change) |
| Top 10 players | `ZREVRANGE leaderboard 0 9 WITHSCORES` |
| A specific player's rank | `ZREVRANK leaderboard player` |
| A specific player's score | `ZSCORE leaderboard player` |
| Players "around" a given player (rank ± 5) | `ZREVRANK` to find their rank, then `ZREVRANGE leaderboard rank-5 rank+5 WITHSCORES` |
| Players within a score range | `ZRANGEBYSCORE leaderboard min max` |
| Total number of ranked players | `ZCARD leaderboard` |

> [!example] "Players around me" query
> ```
> rank = ZREVRANK leaderboard "bob"      # e.g., rank = 42
> ZREVRANGE leaderboard (rank-5) (rank+5) WITHSCORES
> ```
> Two round trips, but each is O(log n) — trivial even for leaderboards with millions of entries.

## 3. Handling tied scores

Sorted Sets break ties **lexicographically by member name** by default, which is rarely the desired tie-breaking rule (usually "whoever reached that score first" should rank higher). A common trick: encode a secondary tiebreaker (like reversed timestamp) into the score itself.

> [!example] Composite score for tie-breaking
> Combine the real score with a timestamp so earlier achievers of the same score naturally sort first:
> ```
> composite_score = real_score * 10^13 + (MAX_TIMESTAMP - achieved_at_timestamp)
> ```
> Storing this single composite number as the ZSet score lets ordinary `ZREVRANGE` calls produce correctly tie-broken results with no extra query logic.

## 4. Multiple leaderboard views

It's common to maintain several parallel Sorted Sets for the same underlying data — a global all-time leaderboard, a weekly leaderboard, a per-region leaderboard — each just a separate key updated by the same write path (e.g., `ZINCRBY leaderboard:global`, `ZINCRBY leaderboard:weekly:2024-W03`, `ZINCRBY leaderboard:region:eu` all in one pipelined batch, see [[Pipelining and Transactions]]).

## 5. Expiring/rotating leaderboards

Time-boxed leaderboards (daily/weekly/seasonal) are naturally modeled as separate keys per period with a TTL slightly longer than the period itself (see [[Keys and Expiry]]), so old leaderboards clean themselves up automatically rather than requiring an explicit cleanup job.

## See also
- [[Sorted Sets]]
- [[Pipelining and Transactions]]
- [[Keys and Expiry]]

#redis #use-cases #leaderboards
