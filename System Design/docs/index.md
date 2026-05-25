# System Design Study Guides

Interview-focused notes on the core building blocks of large-scale systems. Each page follows the same shape: a **mental model**, concrete examples, comparison tables, mermaid diagrams, **interview language** (30/60-second answers), and a **review checklist**.

## How to use this site

- **Studying a topic?** Jump straight to its page from the left nav.
- **Reviewing before an interview?** Read each page's *Final Mental Model* and *Review Checklist* sections first.
- **Following a thread?** Pages cross-link shared concepts to a single canonical home (e.g. replication lives in *Availability & Replication*; consistent hashing in *Sharding & Partitioning*) so you can go deeper without re-reading.

## Suggested study order

1. **Foundations** — how a request travels: networking & request path → HTTP/realtime protocols → a real client→edge→CDN walkthrough.
2. **Databases** — SQL vs NoSQL & ACID → indexing → pagination → sharding → blob storage.
3. **Distributed Systems** — consistency (CAP & PACELC) → availability & replication.
4. **Caching & Scale** — caching patterns → traffic/capacity estimation.
5. **Messaging & APIs** — concurrency → event-driven architecture & messaging → GraphQL.
6. **Resilience** — stability patterns (timeouts, circuit breakers, bulkheads, throttling).
7. **Patterns** — interview-ready coordination patterns: contention (locking, CAS, distributed locks) and multi-step processes (sagas, idempotency, compensation).

## Topic map

| Section | Pages |
|---|---|
| **Foundations** | [Networking & Request Path](foundations/networking-request-path.md) · [HTTP, SSE & WebSockets](foundations/http-and-realtime.md) · [Client → Edge/CDN](foundations/client-edge-cdn.md) |
| **Databases** | [SQL vs NoSQL & ACID](databases/sql-vs-nosql-acid.md) · [Indexing](databases/indexing.md) · [Pagination](databases/pagination.md) · [Sharding & Partitioning](databases/sharding-partitioning.md) · [Blob Storage](databases/blob-storage.md) |
| **Distributed Systems** | [Consistency: CAP & PACELC](distributed-systems/consistency-cap-pacelc.md) · [Availability & Replication](distributed-systems/availability-replication.md) |
| **Caching & Scale** | [Caching Patterns](caching-and-scale/caching-patterns.md) · [Traffic Estimation](caching-and-scale/traffic-estimation.md) |
| **Messaging & APIs** | [Concurrency](messaging-and-apis/concurrency.md) · [Event-Driven Architecture & Messaging](messaging-and-apis/event-driven-and-messaging.md) · [GraphQL](messaging-and-apis/graphql.md) |
| **Resilience** | [Stability Patterns](resilience/stability-patterns.md) |
| **Patterns** | [Contention](patterns/contention.md) · [Multi-step Processes](patterns/multi-step-processes.md) · [Scaling Writes](patterns/scaling-writes.md) · [Scaling Reads](patterns/scaling-reads.md) · [Handling Large Blobs](patterns/large-blobs.md) · [Long-Running Tasks](patterns/long-running-tasks.md) |

## Interactive demos

- [Consistent-hashing ring](assets/consistent-hashing-visual.html) — add/remove nodes, watch keys move (pairs with *Sharding & Partitioning*).
- [Database-choice scenarios](assets/db-choice-interview-scenarios.html) — six worked DB-selection interviews (pairs with *SQL vs NoSQL & ACID*).
- [Wide-column layout](assets/wide-column-visual.html) — partition key + sort key row layout (pairs with *SQL vs NoSQL & ACID*).
