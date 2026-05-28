# 60-Minute Refresher

Goal: the night-before pass. Every core concept compressed to its **mental model**, its **one key trade-off**, **when to use it**, and a **production example** — each linking to the full deep-dive page. Read top to bottom in ~60 minutes, or jump to a weak spot.

!!! tip "How to read this page"
    This is the **refresher track**. Each block is the 80/20 of a deep-dive page. If a block makes you go *"wait, why?"*, follow its link — that's your signal to re-study that topic. If every block lands, you're ready.

<!-- SECTION: table-of-contents -->

## Table of Contents

1. [Foundations](#1-foundations)
2. [Databases](#2-databases)
3. [Distributed Systems](#3-distributed-systems)
4. [Caching & Scale](#4-caching-scale)
5. [Messaging & APIs](#5-messaging-apis)
6. [Resilience](#6-resilience)
7. [Patterns](#7-patterns)
8. [Technology Picks](#8-technology-picks)
9. [The One-Page Cheat Sheet](#9-the-one-page-cheat-sheet)

<!-- SECTION: foundations -->

## 1. Foundations

**Request path** → [deep dive](../foundations/networking-request-path.md)
A request travels client → DNS → CDN/edge → load balancer (L4 or L7) → app server. **L4** balances on IP/port (fast, dumb); **L7** balances on HTTP content (smart, can route by path/header). *Trade-off:* L7 features cost latency. *In production:* AWS ALB is L7, NLB is L4.

**Communication protocols** → [deep dive](../foundations/http-and-realtime.md)
The ladder: **polling** (simple, wasteful) → **long polling** → **SSE** (one-way server push, browser-friendly) → **WebSocket** (full duplex) → **gRPC streaming** (service-to-service). *When to use:* dashboards/notifications → SSE; chat/collab/games → WebSocket; request-response → plain HTTP. *In production:* Slack and WhatsApp use WebSockets; ChatGPT streams tokens over SSE.

**CDN / edge** → [deep dive](../foundations/client-edge-cdn.md)
Cache static assets and cacheable responses close to users. **Pull CDN** fetches from origin on first miss (easy, good for long-tail); **push CDN** you upload ahead of time (good for large, predictable files). *In production:* Cloudflare, Akamai, CloudFront.

<!-- SECTION: databases -->

## 2. Databases

**SQL vs NoSQL** → [deep dive](../databases/sql-vs-nosql-acid.md) · [tech picks](../key-technologies/datastores.md)
Choose by **access pattern and guarantees**, not hype. **SQL** (Postgres/MySQL): relational structure, joins, multi-row ACID transactions, strong consistency. **NoSQL**: document (Mongo), key-value (DynamoDB/Redis), wide-column (Cassandra), graph (Neo4j) — each trades joins/transactions for horizontal scale and a specific access shape. *Rule of thumb:* start relational; reach for NoSQL when one access pattern needs scale a single node can't give. *In production:* Postgres runs most startups far longer than people expect.

**ACID vs BASE**
**ACID** = atomicity, consistency, isolation, durability (correctness-first). **BASE** = basically available, soft state, eventual consistency (availability-first). *Trade-off:* you can't have strong consistency and full availability during a partition (see CAP).

**Indexing** → [deep dive](../databases/indexing.md)
An index trades **write speed + storage** for **read speed**. **B-tree** is the default (range + equality); **hash** for exact match only; **composite** for multi-column filters (order matters — leftmost prefix); **covering** index answers a query from the index alone. *Don't* index low-cardinality columns or over-index write-heavy tables.

**Pagination** → [deep dive](../databases/pagination.md)
**Offset** (`LIMIT/OFFSET`): simple, supports jump-to-page, but slow at deep offsets and unstable under inserts. **Cursor/keyset**: stable and fast at any depth, but no random page access. *Use cursor* for infinite scroll and large datasets. *In production:* every "load more" feed uses cursors.

**Sharding & partitioning** → [deep dive](../databases/sharding-partitioning.md)
Split data horizontally across nodes when one can't hold it or serve it. Shard by a key with even distribution; **consistent hashing** minimizes data movement when nodes change. *The trap:* hot shards (celebrity user, sequential IDs) and cross-shard queries/transactions. *Pick a shard key that spreads load and keeps related data together.*

**Blob storage** → [deep dive](../databases/blob-storage.md)
Big binary files (images, video, ML weights) go in an **object store** (S3); the database holds **metadata + a pointer**. Clients upload directly via **presigned URLs** so bytes never touch your app servers. *In production:* S3 + CloudFront is the default media stack.

<!-- SECTION: distributed-systems -->

## 3. Distributed Systems

**CAP & PACELC** → [deep dive](../distributed-systems/consistency-cap-pacelc.md)
During a network **partition** you must choose **consistency** (reject/stale-block) or **availability** (serve possibly-stale). **PACELC** adds: *else* (no partition), you still trade **latency** vs **consistency**. *Examples:* likes → AP; payments/inventory → CP. *In production:* DynamoDB is tunably AP; Spanner is CP (pays latency for strong consistency).

**Availability & replication** → [deep dive](../distributed-systems/availability-replication.md)
Replicas keep you up when nodes die. **Single-leader** (one writer, many read replicas — simple, has replication lag); **multi-leader** (write anywhere — conflict resolution needed); **leaderless** (quorum reads/writes, Dynamo-style). **RPO** = data you can lose; **RTO** = time to recover. *Trade-off:* sync replication = no data loss but higher write latency; async = fast but a failover can lose the lag window.

<!-- SECTION: caching-scale -->

## 4. Caching & Scale

**Caching patterns** → [deep dive](../caching-and-scale/caching-patterns.md)
The core question: *what can be reused, and how stale can it be?* Read patterns: **cache-aside** (app manages it — the default) and **read-through**. Write patterns: **write-around**, **write-through** (consistent, slower writes), **write-behind** (fast, risk of loss). *Watch for:* stampedes (add TTL jitter + request coalescing), and invalidation — "the hardest problem." *In production:* Redis/Memcached as a look-aside cache in front of the DB.

**Traffic estimation** → [deep dive](../caching-and-scale/traffic-estimation.md)
Clarify → decompose (Fermi) → apply a peak factor (2-10x) → conclude. Compute **only the numbers that change the design**: peak QPS (read vs write), storage/year, bandwidth. *The point isn't precision — it's justifying the architecture* ("50K reads/sec means we cache").

<!-- SECTION: messaging-apis -->

## 5. Messaging & APIs

**Concurrency** → [deep dive](../messaging-and-apis/concurrency.md)
*What can run at once, and what must be coordinated?* **Shared-state** (locks/mutexes/atomics — fast, deadlock risk); **message-passing** (queues/channels — no shared memory, the distributed default). When two requests touch one resource, you need coordination — see [Contention](../patterns/contention.md).

**Event-driven & messaging** → [deep dive](../messaging-and-apis/event-driven-and-messaging.md)
Decouple producers from consumers. **Queue** (SQS/RabbitMQ): work distribution, message consumed once. **Log** (Kafka): replayable, multiple independent consumers, ordered per partition. Delivery is **at-least-once** by default → make consumers **idempotent**. *In production:* Kafka for event streaming/analytics pipelines; SQS for simple task queues.

**GraphQL** → [deep dive](../messaging-and-apis/graphql.md)
Client picks the response shape; one request, no over/under-fetching. *Cost:* caching is harder (POST, dynamic queries), and the N+1 resolver problem (fix with DataLoader batching). *Use when* many clients need different slices of the same graph; *avoid for* simple CRUD where REST is cheaper.

<!-- SECTION: resilience -->

## 6. Resilience

**Stability patterns** → [deep dive](../resilience/stability-patterns.md)
Stop one failure from taking down everything. **Timeout** (never wait forever); **retry + backoff + jitter** (but only for idempotent ops); **circuit breaker** (stop hammering a dead dependency, fail fast); **bulkhead** (isolate resource pools so one drowning dependency can't starve the rest); **throttle/rate-limit** (shed load to protect the core). *In production:* these compose — e.g., Netflix Hystrix popularized the circuit breaker + bulkhead combo.

<!-- SECTION: patterns -->

## 7. Patterns

These are the reusable interview moves. Recognize the trigger, apply the move.

| Pattern | Trigger | The move | Deep dive |
|---|---|---|---|
| **Contention** | Two writers race for one resource | Atomic update → optimistic lock (version/CAS) → pessimistic lock → distributed lock → queue serialization | [Contention](../patterns/contention.md) |
| **Multi-step processes** | One operation spans services/DBs | Idempotency keys + saga (choreography/orchestration) + compensating transactions + outbox | [Multi-step](../patterns/multi-step-processes.md) |
| **Scaling writes** | Writes exceed one node | Batch/buffer → async queue → write sharding → LSM store → CQRS | [Scaling Writes](../patterns/scaling-writes.md) |
| **Scaling reads** | Reads exceed one node | Index → read replica → app cache → distributed cache → CDN → search index → materialized view | [Scaling Reads](../patterns/scaling-reads.md) |
| **Large blobs** | Big files move client↔storage | Presigned URLs → multipart → resumable → CDN delivery → async post-processing | [Large Blobs](../patterns/large-blobs.md) |
| **Long-running tasks** | Work too slow for one request | Split accept from execute: queue + worker + job ID, notify via polling/SSE/webhook | [Long-Running Tasks](../patterns/long-running-tasks.md) |

<!-- SECTION: technology-picks -->

## 8. Technology Picks

The fast "when do I pick what" — full tables in the [Technology Cheat Sheet](../key-technologies/index.md).

| Need | Default pick | Reach for instead when… |
|---|---|---|
| Relational data + transactions | **PostgreSQL** | Global strong consistency at scale → **Spanner/CockroachDB** |
| Massive scale, key-based access | **DynamoDB** (managed) | Self-hosted, write-heavy, multi-region → **Cassandra** |
| Flexible documents | **MongoDB** | — |
| Cache / ephemeral state | **Redis** (rich types, persistence) | Pure simple cache, multi-threaded → **Memcached** |
| Event streaming / replay | **Kafka** | Simple managed task queue → **SQS**; complex routing → **RabbitMQ** |
| Full-text / fuzzy search | **Elasticsearch** | — |
| Large files / media | **S3 + CloudFront** | — |
| Coordination / leader election | **etcd** (or ZooKeeper) | Service discovery + config → **Consul** |

<!-- SECTION: cheat-sheet -->

## 9. The One-Page Cheat Sheet

> **Read-heavy?** Cache (cache-aside) + read replicas + CDN. Watch for stampedes (jitter + coalescing).
>
> **Write-heavy?** Batch → async queue → shard by an even key → LSM-friendly store. Watch for hot shards.
>
> **Two writers, one resource?** Atomic update first; escalate to optimistic → pessimistic → distributed lock → queue.
>
> **Slow operation?** Split accept from execute: 202 + job ID, worker pool, notify via SSE/webhook. Make workers idempotent.
>
> **Operation spans services?** Saga + idempotency keys + compensating transactions + outbox for the dual-write.
>
> **Big files?** Object store + metadata pointer; presigned URLs so bytes skip your app servers; CDN for delivery.
>
> **Need strong consistency?** Single-leader / CP store / transactions. **Can tolerate stale?** Replicas / AP store / eventual consistency — and say so out loud.
>
> **"Scale it 10x"?** Walk the request path, name the *specific* bottleneck (hot key, fan-out, replication lag, single point), apply the matching escalation ladder.
>
> **Every component you add → name its failure mode.** Cache → stampede. Queue → backlog/poison message. Replica → lag. Shard → hot partition. Lock → contention/deadlock.

That last line is the whole game: **name the why, name the failure mode, name the cost.** If you can do that for every box you draw, you're interviewing at senior+.
