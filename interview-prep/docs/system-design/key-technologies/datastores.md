# Datastores

Goal: pick the right database in an interview and defend it by access pattern. Covers Postgres, MySQL, DynamoDB, Cassandra, MongoDB, and the NewSQL options. A full read takes about 12 minutes. For the *mechanics* of ACID/BASE and sharding, see [SQL vs NoSQL & ACID](../databases/sql-vs-nosql-acid.md) and [Sharding & Partitioning](../databases/sharding-partitioning.md).

<!-- SECTION: table-of-contents -->

## Table of Contents

1. [Mental Model](#1-mental-model)
2. [The Families](#2-the-families)
3. [Selection Table](#3-selection-table)
4. [Relational: Postgres vs MySQL](#4-relational-postgres-vs-mysql)
5. [Wide-Column / KV: DynamoDB vs Cassandra](#5-wide-column-kv-dynamodb-vs-cassandra)
6. [Document: MongoDB](#6-document-mongodb)
7. [NewSQL: Spanner & CockroachDB](#7-newsql-spanner-cockroachdb)
8. [Interview Language](#8-interview-language)
9. [Review Checklist](#9-review-checklist)

<!-- SECTION: mental-model -->

## 1. Mental Model

> **The database is chosen by how data is accessed, not by how big it is.** Joins and multi-row transactions → relational. Lookups by a known key at massive write volume → wide-column/KV. Nested, evolving documents → document store. Global strong consistency at scale → NewSQL. "It's a lot of data" alone doesn't pick the database — the access pattern does.

Mental shortcut: **relational by default; switch only when one access pattern needs scale or a shape a single relational node can't give.**

<!-- SECTION: families -->

## 2. The Families

| Family | Data shape | Killer feature | Weakness | Examples |
|---|---|---|---|---|
| **Relational** | Tables, rows, FKs | Joins + ACID transactions | Hard to scale writes horizontally | Postgres, MySQL |
| **Key-value** | Key → opaque value | O(1) lookup, huge scale | No queries beyond the key | DynamoDB, Redis |
| **Wide-column** | Partition key + clustering key → columns | Write-optimized, linear scale | Must model around access pattern up front | Cassandra, ScyllaDB, Bigtable |
| **Document** | JSON-like documents | Flexible schema, nested data | Joins are awkward; consistency is per-doc | MongoDB, Couchbase |
| **Graph** | Nodes + edges | Relationship traversal | Niche; scales poorly | Neo4j |
| **NewSQL** | Tables + SQL | Horizontal scale **and** strong consistency | Higher latency, complexity, cost | Spanner, CockroachDB |

<!-- SECTION: selection-table -->

## 3. Selection Table

| If you need… | Pick | Because | In production |
|---|---|---|---|
| Transactions across rows, joins, ad-hoc queries | **PostgreSQL** | Mature, correct, flexible; handles surprising scale | Most startups; Instagram (early + still core) |
| Same, with heavy read replicas + simpler ops | **MySQL** | Battle-tested replication, huge ecosystem | YouTube, Facebook (core) |
| Key-based access, predictable single-digit-ms latency, managed | **DynamoDB** | Auto-sharded, serverless, scales writes linearly | Amazon cart, Lyft, Snapchat |
| Write-heavy, multi-region, no single leader, self-hosted | **Cassandra** | Leaderless, tunable consistency, linear write scale | Discord messages, Netflix viewing history, Instagram |
| Flexible/nested documents, secondary-index queries | **MongoDB** | Schema flexibility + rich query API | eBay, content/CMS systems |
| SQL semantics + horizontal scale + strong consistency globally | **Spanner / CockroachDB** | Distributed transactions with external consistency | Google (Spanner), financial/global apps |

<!-- SECTION: relational -->

## 4. Relational: Postgres vs MySQL

Both are excellent; the choice rarely loses points either way. Differences worth a sentence:

| | PostgreSQL | MySQL |
|---|---|---|
| **Strengths** | Richer types (JSONB, arrays, GIS), stricter SQL compliance, advanced indexing (GIN/GiST), powerful query planner | Simpler, very fast for read-heavy simple queries, mature replication, ubiquitous |
| **Reach for it when** | Complex queries, JSON + relational mix, geospatial, correctness-critical | Read-heavy web apps, huge replica fleets, operational familiarity |

> **Why default to Postgres:** it stretches furthest before you need something specialized — JSONB gives you document-store flexibility, PostGIS gives geospatial, full-text search is built in. You can often avoid adding a second system entirely.

<!-- SECTION: wide-column -->

## 5. Wide-Column / KV: DynamoDB vs Cassandra

Both scale writes horizontally with no single leader bottleneck. The split is **managed vs self-hosted** and **single-cloud vs multi-region-flexible**.

| | DynamoDB | Cassandra |
|---|---|---|
| **Ops model** | Fully managed (AWS) | Self-hosted / DataStax |
| **Consistency** | Eventual or strong (per-read setting) | Tunable per query (QUORUM, ONE, ALL) |
| **Best for** | AWS shops wanting zero ops | Multi-region active-active, write-heavy, cloud-agnostic |
| **Watch out** | Cost at scale; hot partitions; 400KB item limit | Operational burden; you must model tables per query up front |

> **The shared rule:** in both, you design the table around the query. There are no joins and no ad-hoc filters — the partition key *is* your access pattern. *In production:* Discord stores trillions of messages in Cassandra/ScyllaDB partitioned by channel; you can only read efficiently the way you partitioned.

<!-- SECTION: document -->

## 6. Document: MongoDB

Stores JSON-like documents; schema can vary per document. Good when the data is naturally a nested object (a product with variants, a user with embedded settings) and you mostly read/write whole documents.

- **Strength:** schema flexibility, fast iteration, rich secondary indexes and aggregation pipeline.
- **Weakness:** joins (`$lookup`) are awkward and slow; transactions exist but are not its strength; easy to model yourself into denormalization pain.
- **Reach for it when:** the access unit is a self-contained document and the schema evolves often. *Avoid* when you have highly relational data with many-to-many joins — that's relational's job.

<!-- SECTION: newsql -->

## 7. NewSQL: Spanner & CockroachDB

These give you **SQL + ACID transactions + horizontal scale + strong consistency across regions** — the thing CAP says is expensive. They pay for it in latency (cross-region consensus) and cost/complexity.

- **Reach for them when:** you genuinely need global strong consistency at scale — financial ledgers, inventory, global user accounts — and can't accept the eventual consistency of a Dynamo-style store.
- **Don't reach for them when:** a single-region Postgres with replicas meets your SLOs. They're the right answer to a real constraint, and over-engineering otherwise. *In production:* Google uses Spanner for AdWords/ads infrastructure; CockroachDB powers globally-distributed fintech and SaaS.

<!-- SECTION: interview-language -->

## 8. Interview Language

- *"Access is by a single known key at very high write volume, so DynamoDB — relational would bottleneck on the primary's write path."*
- *"There are real joins and multi-row transactions here, so I'd stay relational; I'd scale reads with replicas and a cache before considering sharding."*
- *"I need active-active multi-region writes, which is Cassandra's leaderless model; I'd tune to QUORUM where I need read-your-writes."*
- *"This needs global strong consistency on money, so I'd justify Spanner despite the latency cost — eventual consistency isn't acceptable for a ledger."*

<!-- SECTION: review-checklist -->

## 9. Review Checklist

- [ ] Can you name the access pattern that justifies each family?
- [ ] Postgres vs MySQL — can you give one differentiating reason?
- [ ] DynamoDB vs Cassandra — managed/AWS vs self-hosted/multi-region — can you state it?
- [ ] Do you know why wide-column stores force you to model around the query?
- [ ] Can you say when NewSQL is justified *and* when it's over-engineering?
- [ ] Do you default to relational and upgrade only on a real constraint?
