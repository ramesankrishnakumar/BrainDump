# Coordination Services

Goal: know what ZooKeeper, etcd, and Consul are for, when a design actually needs one, and how to justify the pick. A full read takes about 7 minutes.

<!-- SECTION: table-of-contents -->

## Table of Contents

1. [Mental Model](#1-mental-model)
2. [What They Do](#2-what-they-do)
3. [ZooKeeper vs etcd vs Consul](#3-zookeeper-vs-etcd-vs-consul)
4. [When You Actually Need One](#4-when-you-actually-need-one)
5. [The Consensus Underneath](#5-the-consensus-underneath)
6. [Interview Language](#6-interview-language)
7. [Review Checklist](#7-review-checklist)

<!-- SECTION: mental-model -->

## 1. Mental Model

> **A coordination service is a small, strongly-consistent (CP) store that a cluster uses to agree on things** — who's the leader, which nodes are alive, what the current config is, who holds a lock. It is not a general database; it's a low-volume source of truth for *metadata about the cluster itself*, backed by a consensus algorithm so all nodes see the same answer.

Mental shortcut: **need many nodes to agree on one fact (leader, membership, config, lock) → coordination service. Need to store application data → a real database.**

<!-- SECTION: what-they-do -->

## 2. What They Do

| Job | What it means | Example |
|---|---|---|
| **Leader election** | Pick exactly one coordinator among peers | Kafka controller, primary DB selection |
| **Service discovery** | Track which instances are alive and where | Microservices finding each other |
| **Configuration management** | Push consistent config to all nodes | Feature flags, dynamic settings |
| **Distributed locks** | Grant a lock no two nodes can hold at once | Ensuring one job runs cluster-wide |
| **Membership / health** | Know which nodes joined/left/died | Cluster rebalancing |

<!-- SECTION: comparison -->

## 3. ZooKeeper vs etcd vs Consul

| | ZooKeeper | etcd | Consul |
|---|---|---|---|
| **Consensus** | ZAB | Raft | Raft |
| **Data model** | Hierarchical znodes | Flat key-value | KV + rich service catalog |
| **Sweet spot** | Classic big-data ecosystem | Cloud-native / Kubernetes | Service discovery + health + config |
| **Ecosystem** | Kafka (legacy), Hadoop, HBase | Kubernetes (its backing store), CoreOS | HashiCorp stack, multi-DC |
| **Interface** | Custom client | gRPC / HTTP | HTTP/DNS |

- **etcd** — the modern default; it's what **Kubernetes** uses to store all cluster state. Reach for it in cloud-native systems.
- **ZooKeeper** — battle-tested, dominant in the big-data world. (Kafka historically used it; newer Kafka uses its own Raft-based KRaft instead.)
- **Consul** — when you want service discovery + health checking + KV config + multi-datacenter together out of the box.

<!-- SECTION: when-needed -->

## 4. When You Actually Need One

> **Most interview answers do NOT need a named coordination service — and saying so is a senior signal.** A distributed lock can be a Redis `SET NX`; leader election can be a database row with a lease. Reach for ZooKeeper/etcd only when you genuinely need strong, consensus-backed coordination across many nodes.

You need one when:

- You're **building the infrastructure** (a Kafka-like broker, a custom distributed database, a scheduler) that itself must elect leaders and track membership.
- You need **strict correctness** on coordination that a Redis lock's edge cases (clock skew, failover) can't guarantee.

You **don't** need one when a single managed database, a Redis lock, or your cloud's built-in service discovery already solves it. Naming etcd for a simple web app is over-engineering. See the lock escalation ladder in [Contention](../patterns/contention.md).

<!-- SECTION: consensus -->

## 5. The Consensus Underneath

These services are strongly consistent because they run a **consensus algorithm** (Raft or ZAB/Paxos-family) over a small cluster (typically 3 or 5 nodes). A write is committed once a **majority (quorum)** agrees, so the system survives a minority of node failures while every node converges on the same value.

- **Why odd numbers (3, 5):** a quorum is ⌊n/2⌋+1; an odd count maximizes fault tolerance per node and avoids split-brain ties.
- **The trade-off:** consensus is CP — during a partition the minority side stops accepting writes to preserve consistency. That's the right choice for coordination metadata, where being *correct* matters more than being *available*.

<!-- SECTION: interview-language -->

## 6. Interview Language

- *"This needs cluster-wide leader election with strict correctness, so a Raft-backed service like etcd — a Redis lock has failover edge cases I don't want here."*
- *"I don't need a coordination service for this; a database lease row or a Redis lock is sufficient at this scale."*
- *"Kubernetes already gives me service discovery and etcd-backed state, so I'd lean on the platform rather than running ZooKeeper myself."*
- *"Coordination services are CP — during a partition the minority stops accepting writes, which is exactly what I want for leader election."*

<!-- SECTION: review-checklist -->

## 7. Review Checklist

- [ ] Can you explain a coordination service as a small CP metadata store for cluster agreement?
- [ ] Name the four core jobs (election, discovery, config, locks)?
- [ ] etcd vs ZooKeeper vs Consul — one differentiator each?
- [ ] Can you explain why most designs *don't* need one, and what to use instead?
- [ ] Why quorum + odd node counts, and why CP over AP for coordination?
