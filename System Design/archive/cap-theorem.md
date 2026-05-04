````markdown
# CAP Theorem Study Guide

## 1. What is CAP Theorem?

CAP theorem says that in a distributed system, when a **network partition** happens, the system must choose between:

- **Consistency**
- **Availability**

The three CAP properties are:

| Letter | Meaning | Simple Explanation |
|---|---|---|
| **C** | Consistency | Every user sees the latest correct data |
| **A** | Availability | Every request gets a response, even if data may be stale |
| **P** | Partition Tolerance | The system keeps operating even when parts of it cannot communicate |

In real distributed systems, **network partitions can happen**, so partition tolerance is usually required.

So the practical CAP question is:

> When the network breaks, should the system protect correctness or keep responding?

---

## 2. What does “Partition” mean?

A **partition** means the system is split into groups that are still running, but cannot communicate with each other.

Example:

```text
USA Region        ❌ network issue ❌        Europe Region
Likes DB US                                  Likes DB EU
Users in USA                                 Users in Europe
````

The USA region is not necessarily down.
The Europe region is not necessarily down.
But they cannot sync with each other.

So a partition is not always:

```text
User → Service ❌ unreachable
```

It is usually more like:

```text
User → Local Region ✅ works
Local Region → Other Region ❌ broken
```

---

## 3. Likes Example

Imagine an article has:

```text
Likes = 100
```

A network partition happens between USA and Europe.

USA users add 5 likes:

```text
USA DB = 105
```

Europe users add 3 likes:

```text
Europe DB = 103
```

During the partition:

```text
USA users may see 105 likes
Europe users may see 103 likes
```

After the network recovers:

```text
USA and Europe sync
Final count = 108
```

This is acceptable for likes because a slightly stale count is usually okay.

---

## 4. AP Choice: Availability Over Consistency

For likes, social feeds, views, analytics, and recommendations, systems often choose **AP**.

That means:

```text
User clicks Like
→ Local Like Service accepts request
→ Like is stored locally
→ Other regions may not see it immediately
→ Data is fixed later through replication/reconciliation
```

This gives users a fast, available experience.

The tradeoff:

```text
Users get responses,
but some data may be temporarily stale.
```

---

## 5. CP Choice: Consistency Over Availability

For payments, inventory, banking, booking, and order state transitions, systems often choose **CP**.

That means:

```text
If the system cannot confirm the latest correct state,
it may reject, delay, or timeout the request.
```

Example: bank balance

```text
Balance = $100
```

During a partition, USA and Europe both think the balance is $100.

If both allow withdrawals independently:

```text
USA withdrawal = $80
Europe withdrawal = $80
Total withdrawn = $160
```

That is incorrect.

So a CP system may do this instead:

```text
Cannot confirm latest balance
→ reject or delay transaction
```

The tradeoff:

```text
Data stays correct,
but some users may get errors or timeouts.
```

---

## 6. Applying CAP in System Design

Do not say:

```text
The whole system is CP
```

or:

```text
The whole system is AP
```

Instead, apply CAP **feature by feature**.

Example: E-commerce system

| Feature               | CAP Choice | Reason                                     |
| --------------------- | ---------- | ------------------------------------------ |
| Product catalog       | AP         | Slightly stale product info is okay        |
| Reviews and ratings   | AP         | Can be updated later                       |
| Likes/views           | AP         | Stale counts are acceptable                |
| Cart                  | Usually AP | Temporary sync delay is acceptable         |
| Inventory reservation | CP         | Avoid overselling                          |
| Payment               | CP         | Avoid double-charging or incorrect charges |
| Order state           | Usually CP | Users need correct order status            |

---

## 7. How to Decide CP vs AP

Ask these questions:

### Can users tolerate stale data?

If yes, AP may be okay.

Examples:

```text
likes
views
recommendations
search results
analytics
notifications
```

### Can the system tolerate conflicting writes?

If no, choose CP.

Examples:

```text
payments
bank balances
seat booking
inventory reservation
permissions
order state transitions
```

### Is it better to fail than return wrong data?

If yes, choose CP.

### Is it better to return stale data than fail?

If yes, choose AP.

---

## 8. Common System Design Language

### For AP systems

Use terms like:

```text
eventual consistency
async replication
local writes
conflict resolution
reconciliation jobs
idempotent writes
message queues
```

Example answer:

> For likes, I would choose availability over consistency. Each region can accept likes locally, and counts can be reconciled asynchronously. Users may temporarily see stale counts, but the system remains responsive.

---

### For CP systems

Use terms like:

```text
strong consistency
leader-based writes
quorum reads/writes
transactions
distributed locks
consensus
fail fast
```

Example answer:

> For payments, I would choose consistency over availability. If the system cannot confirm the latest account state, it should reject or delay the transaction rather than risk double-spending or incorrect charges.

---

## 9. Key Mental Model

Partition tolerance means:

```text
Parts of the system cannot talk to each other,
but they may still be serving users locally.
```

For AP:

```text
Keep accepting requests
Allow temporary inconsistency
Repair later
```

For CP:

```text
Block or reject unsafe requests
Preserve correctness
Sacrifice some availability
```

---

## 10. Interview-Ready Summary

> CAP theorem helps decide what a distributed system should do during a network partition. Since partitions can happen in real systems, the practical tradeoff is between consistency and availability. For non-critical features like likes, views, and recommendations, I would choose availability and allow eventual consistency. For critical features like payments, inventory reservation, and banking, I would choose consistency and reject or delay requests if the system cannot safely coordinate.

```
```


Below is a **realistic real-system example**: a **global article “likes” service** built using a **Dynamo/Cassandra-style AP architecture**.

Amazon’s Dynamo paper is a classic real-world example of a highly available key-value store that sacrifices strict consistency under some failures, using eventual consistency and conflict resolution. Apache Cassandra’s own docs also describe Cassandra as prioritizing **Availability + Partition Tolerance**, compromising consistency to some extent. ([allthingsdistributed.com][1])

---

# Example: Global Article Likes System

## Goal

Users can like an article from anywhere in the world.

Example:

```text
Article ID: article_123
Current likes: 100
```

Users in the USA and Europe should be able to click **Like** even if the network between regions is broken.

For likes, we prefer:

```text
Availability > Strong Consistency
```

Because showing `101` likes instead of `103` for a short time is acceptable.

---

# Architecture Diagram

```text
                       ┌─────────────────────┐
                       │     Global DNS /    │
                       │   Traffic Router    │
                       └──────────┬──────────┘
                                  │
              ┌───────────────────┴───────────────────┐
              │                                       │
              ▼                                       ▼
┌───────────────────────────┐           ┌───────────────────────────┐
│        USA Region          │           │       Europe Region        │
│                            │           │                            │
│  ┌───────────────┐         │           │  ┌───────────────┐         │
│  │  Like API US  │         │           │  │  Like API EU  │         │
│  └───────┬───────┘         │           │  └───────┬───────┘         │
│          │                 │           │          │                 │
│          ▼                 │           │          ▼                 │
│  ┌───────────────┐         │           │  ┌───────────────┐         │
│  │ Likes DB US   │         │           │  │ Likes DB EU   │         │
│  │ article_123   │         │           │  │ article_123   │         │
│  │ likes = 105   │         │           │  │ likes = 103   │         │
│  └───────┬───────┘         │           │  └───────┬───────┘         │
│          │                 │           │          │                 │
│          ▼                 │           │          ▼                 │
│  ┌───────────────┐         │           │  ┌───────────────┐         │
│  │ Event Log US  │         │           │  │ Event Log EU  │         │
│  └───────┬───────┘         │           │  └───────┬───────┘         │
└──────────┼─────────────────┘           └──────────┼─────────────────┘
           │                                        │
           └─────────────── async sync ─────────────┘

                    ❌ During partition, this link breaks ❌
```

---

# Normal Flow: No Partition

A USA user likes the article.

```text
USA user clicks Like
→ Request goes to Like API US
→ Like API US writes to Likes DB US
→ Like API US appends event to Event Log US
→ Async replication sends event to Europe
→ Likes DB EU eventually updates too
```

After sync:

```text
USA DB:    article_123 likes = 101
Europe DB: article_123 likes = 101
```

Both regions eventually agree.

---

# Partition Happens

Now suppose the network between USA and Europe breaks.

```text
USA Region              ❌ network partition ❌              Europe Region
```

Important: both regions are still alive.

USA users can reach USA services.

Europe users can reach Europe services.

But USA and Europe cannot sync with each other.

---

# What Happens When We Choose Availability?

## USA user likes the article

```text
USA user clicks Like
→ Like API US is reachable
→ Like is stored in Likes DB US
→ Event is stored in Event Log US
→ User gets success response
```

USA count becomes:

```text
USA DB: article_123 likes = 105
```

## Europe user likes the same article

```text
Europe user clicks Like
→ Like API EU is reachable
→ Like is stored in Likes DB EU
→ Event is stored in Event Log EU
→ User gets success response
```

Europe count becomes:

```text
Europe DB: article_123 likes = 103
```

During the partition:

```text
USA users may see:    105 likes
Europe users may see: 103 likes
```

This is **inconsistent**, but the system is still **available**.

That is AP behavior.

---

# Why This Is Availability Over Consistency

The system does **not** stop USA users from liking just because Europe cannot be reached.

It also does **not** stop Europe users from liking just because USA cannot be reached.

Instead, each region accepts local writes.

```text
Partition happens
→ Keep serving local users
→ Store writes locally
→ Allow temporary disagreement
→ Repair later
```

That is exactly how we choose **Availability over Consistency**.

---

# How the System Is Architected to Support This

## 1. Local writes per region

Each region has its own local database.

```text
USA users write to USA DB
Europe users write to Europe DB
```

The user does not need to wait for all regions to confirm the write.

This keeps latency low and availability high.

---

## 2. Async replication

Cross-region syncing happens asynchronously.

```text
USA Event Log ───── async replication ─────> Europe
Europe Event Log ── async replication ─────> USA
```

If the network is healthy, replication happens quickly.

If the network is broken, events wait locally.

---

## 3. Durable event log

Every like is stored as an event.

Instead of only storing:

```text
article_123 likes = 105
```

we store events like:

```text
user_1 liked article_123 at time T1 in USA
user_2 liked article_123 at time T2 in Europe
user_3 liked article_123 at time T3 in USA
```

This helps reconciliation later.

---

## 4. Idempotent writes

The system should avoid double-counting the same like.

A like event can have a unique ID:

```text
like_id = user_id + article_id
```

So if the same event is replayed twice during recovery, the system knows:

```text
Already processed this like
→ Do not count it again
```

This is called **idempotency**.

---

## 5. Reconciliation after partition heals

When USA and Europe can communicate again:

```text
USA sends missing like events to Europe
Europe sends missing like events to USA
```

Then both regions merge the events.

Final state:

```text
USA had 5 new likes
Europe had 3 new likes

Final count = old count + 5 + 3
Final count = 108
```

Eventually:

```text
USA DB:    article_123 likes = 108
Europe DB: article_123 likes = 108
```

This is **eventual consistency**.

---

# Detailed Write Path

```text
1. User clicks Like
2. Request goes to nearest region
3. Like API validates request
4. Like API writes event locally
5. Local DB updates local count
6. User receives success
7. Background replication sends event to other regions
8. Other regions apply event later
```

The key AP decision is step 6:

```text
Return success after local write,
not after global replication.
```

A CP system would wait for stronger coordination.
An AP system does not.

---

# During Partition: Timeline Example

Initial state:

```text
Global likes = 100
```

Partition starts.

```text
USA and Europe cannot sync.
```

USA activity:

```text
USA receives 5 likes
USA local count = 105
```

Europe activity:

```text
Europe receives 3 likes
Europe local count = 103
```

During partition:

```text
USA users see 105
Europe users see 103
```

Partition ends.

```text
USA sends 5 events to Europe
Europe sends 3 events to USA
```

After reconciliation:

```text
USA users see 108
Europe users see 108
```

---

# Why This Is Okay for Likes

Likes are usually not business-critical.

It is acceptable if users temporarily see:

```text
105 likes
```

instead of:

```text
108 likes
```

The user experience remains good because clicking like still works.

This is very different from payments, where temporary inconsistency could create serious problems.

---

# What This Architecture Would Not Be Good For

This same AP design would be dangerous for:

```text
bank withdrawals
inventory reservation
seat booking
payment confirmation
account balance updates
```

Example problem:

```text
USA thinks 1 ticket is available
Europe also thinks 1 ticket is available

Both regions sell it
→ oversold ticket
```

For that type of system, we usually prefer **Consistency over Availability**.

---

# Interview-Ready Explanation

You can say:

> For a global likes system, I would choose availability over consistency. Each region accepts likes locally and stores them in a durable local event log. Cross-region replication happens asynchronously. During a network partition, USA and Europe may show different like counts, but users can still like posts. Once the partition heals, background reconciliation exchanges missing events and updates the final count. This gives us high availability and eventual consistency, which is acceptable because likes can tolerate stale counts.

---

# One-Line Mental Model

```text
AP design = accept writes locally now, sync globally later.
```

[1]: https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf?utm_source=chatgpt.com "Dynamo: Amazon’s Highly Available Key-value Store"
