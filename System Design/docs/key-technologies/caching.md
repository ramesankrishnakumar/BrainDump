# Caching Technologies

Goal: choose and defend a cache technology, and know the operational gotchas interviewers probe. Covers Redis vs Memcached, data structures, persistence, and clustering. A full read takes about 8 minutes. For *caching patterns* (cache-aside, write-through, invalidation), see [Caching Patterns](../caching-and-scale/caching-patterns.md).

<!-- SECTION: table-of-contents -->

## Table of Contents

1. [Mental Model](#1-mental-model)
2. [Redis vs Memcached](#2-redis-vs-memcached)
3. [What Redis Gives You Beyond a Cache](#3-what-redis-gives-you-beyond-a-cache)
4. [Persistence & Durability](#4-persistence-durability)
5. [Clustering & Scaling](#5-clustering-scaling)
6. [Operational Gotchas](#6-operational-gotchas)
7. [Interview Language](#7-interview-language)
8. [Review Checklist](#8-review-checklist)

<!-- SECTION: mental-model -->

## 1. Mental Model

> **Redis is the default; Memcached is the exception.** They both serve sub-millisecond in-memory reads. Redis wins because it does far more — rich data structures, persistence, pub/sub, atomic operations, clustering. Reach for Memcached only when you want a dead-simple, multi-threaded, pure LRU cache and nothing else.

Mental shortcut: **need only a cache → either; need anything else (counters, locks, queues, sorted sets, persistence) → Redis.**

<!-- SECTION: redis-vs-memcached -->

## 2. Redis vs Memcached

| | Redis | Memcached |
|---|---|---|
| **Data types** | Strings, hashes, lists, sets, sorted sets, bitmaps, HyperLogLog, streams, geo | Strings/blobs only |
| **Threading** | Single-threaded core (very fast; predictable) | Multi-threaded (uses many cores per node) |
| **Persistence** | Optional (RDB snapshots, AOF log) | None — purely ephemeral |
| **Replication / HA** | Yes (replicas + Sentinel + Cluster) | No native replication |
| **Atomic ops** | Rich (INCR, SETNX, Lua scripts, transactions) | Basic (incr/decr) |
| **Use it for** | Cache + counters + leaderboards + rate limiting + locks + queues | Simple, huge, multi-core look-aside cache |

> **Why Redis dominates interviews:** most "design X" answers need *more* than a cache — a rate limiter needs atomic INCR, a leaderboard needs sorted sets, a distributed lock needs SET NX, a feed needs lists. Redis covers all of these, so naming it keeps your options open. *In production:* Redis backs Twitter timelines, GitHub job queues, and countless rate limiters.

<!-- SECTION: beyond-cache -->

## 3. What Redis Gives You Beyond a Cache

These show up constantly in design questions — knowing the mapping is a fast senior signal:

| Need | Redis primitive | Shows up in |
|---|---|---|
| Rate limiting | `INCR` + `EXPIRE`, or token bucket via Lua | [Rate Limiter](../practice-questions/rate-limiter.md) |
| Leaderboard / top-K | Sorted set (`ZADD`/`ZRANGE`) | Gaming, [Scaling Writes](../patterns/scaling-writes.md) |
| Distributed lock | `SET key val NX PX ttl` (Redlock) | [Contention](../patterns/contention.md) |
| Pub/sub fan-out | `PUBLISH`/`SUBSCRIBE` | Live notifications, [Long-Running Tasks](../patterns/long-running-tasks.md) |
| Lightweight queue / stream | Lists or Redis Streams | Job queues |
| Session store | Hash + TTL | Stateless app servers |
| Deduplication / set membership | Sets, HyperLogLog (approx counts) | Unique visitor counts |

<!-- SECTION: persistence -->

## 4. Persistence & Durability

Redis is in-memory but can persist:

- **RDB** — point-in-time snapshots. Compact, fast restart, but you lose writes since the last snapshot.
- **AOF** — append-only log of every write. More durable (configurable fsync), larger, slower restart.
- **Trade-off:** RDB for fast recovery + acceptable loss; AOF (or both) when you need minimal data loss. *Caveat for interviews:* even with persistence, **treat Redis as a cache, not a system of record** unless you've explicitly designed for durability — the database remains the source of truth.

<!-- SECTION: clustering -->

## 5. Clustering & Scaling

- **Replication:** one primary, N read replicas. Replicas serve reads and enable failover.
- **Sentinel:** monitors and automates failover (promotes a replica when the primary dies).
- **Cluster mode:** shards the keyspace across nodes via hash slots (16384 slots). Scales beyond one node's memory; multi-key ops must hit the same slot (use hash tags).
- *Trade-off:* clustering adds the cross-slot constraint and rebalancing complexity — only use it when one node's memory/throughput is genuinely the limit.

<!-- SECTION: gotchas -->

## 6. Operational Gotchas

Interviewers love these — name them proactively:

| Gotcha | What happens | Mitigation |
|---|---|---|
| **Cache stampede / thundering herd** | A hot key expires; thousands of requests hit the DB at once | TTL jitter + request coalescing (single-flight) + optional lock-on-miss |
| **Hot key** | One key gets disproportionate traffic, saturating one node | Client-side local cache tier, key replication, or split the key |
| **Eviction surprise** | Memory full → keys evicted under `maxmemory` policy | Pick the right policy (`allkeys-lru` for cache); size for the working set |
| **Cache down** | Look-aside cache dies → full read load falls on the DB | Circuit breaker + graceful degradation; never let the cache be a hard dependency |
| **Stale data** | Cache and DB diverge after a write | Choose an invalidation strategy deliberately (see [Caching Patterns](../caching-and-scale/caching-patterns.md)) |

<!-- SECTION: interview-language -->

## 7. Interview Language

- *"I'll use Redis — not just for caching but because I also need atomic counters for rate limiting and a sorted set for the leaderboard."*
- *"It's a pure look-aside cache with no extra needs, so Memcached's multi-threaded simplicity is fine — but I'd default to Redis to keep options open."*
- *"The risk with a hot key expiring is a stampede onto the DB, so I'd add TTL jitter and single-flight coalescing."*
- *"Redis is a cache here, not the source of truth — if it's down we degrade to the database behind a circuit breaker, we don't fail."*

<!-- SECTION: review-checklist -->

## 8. Review Checklist

- [ ] Can you state Redis vs Memcached in one sentence (rich + persistent vs simple + multi-threaded)?
- [ ] Can you map 4+ design needs to Redis primitives (counter, lock, leaderboard, pub/sub)?
- [ ] Do you know RDB vs AOF and that Redis is still a cache, not a source of truth?
- [ ] Can you explain cache stampede and its fix without prompting?
- [ ] Do you treat the cache as a soft dependency with graceful degradation?
