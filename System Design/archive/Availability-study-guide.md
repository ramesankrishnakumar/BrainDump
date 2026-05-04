Think of **availability patterns** as ways a system keeps working when machines, networks, or data centers fail.

Let’s use a real-world architecture: **an online food delivery app** like DoorDash/Uber Eats.

Core services might include:

```text
Mobile App
   ↓
API Gateway
   ↓
Order Service ─── Payment Service
   ↓
Restaurant Service
   ↓
Delivery Dispatch Service
   ↓
Databases / Caches / Message Queues
```

The business goal is: even if one server, database, or region fails, users can still browse restaurants, place orders, and track deliveries.

---

## 1. Fail-over

**Fail-over** means: when the active system fails, another system takes over.

Imagine the **Order Service database** has a primary database in Virginia:

```text
Order DB - Primary
Virginia
```

And a standby database in Ohio:

```text
Order DB - Standby
Ohio
```

Normally, all writes go to Virginia. If Virginia goes down, traffic switches to Ohio.

```text
Before failure:

Order Service → Primary DB

After failure:

Order Service → Standby DB
```

In a food delivery app, fail-over helps avoid a situation where users cannot place orders just because one database or server died.

There are two common styles:

**Active-passive fail-over**: one system handles traffic, the backup waits.

```text
Primary: active
Backup: passive
```

**Active-active fail-over**: multiple systems handle traffic at the same time, and if one fails, the others continue.

```text
Region A: active
Region B: active
Region C: active
```

The tradeoff is that fail-over improves availability, but it introduces complexity. You need health checks, traffic routing, data synchronization, and a plan for what happens to writes that happened right before the failure.

---

## 2. Replication

**Replication** means keeping copies of data or services in multiple places.

For example, the food delivery app might replicate restaurant data:

```text
Restaurant DB
   ├── Copy in US-East
   ├── Copy in US-West
   └── Copy in Europe
```

This helps with both **availability** and **performance**.

If the US-East database fails, the app can read from US-West. If a user is in California, the app can read nearby data from US-West instead of querying a faraway region.

Replication is used for databases, caches, search indexes, files, message queues, and even full services.

The major question is: **how do we keep copies consistent?**

For example, suppose a restaurant changes its hours from 9 PM closing to 10 PM closing. How quickly do all replicas learn that?

That leads to different replication patterns.

---

# Types of replication

## 2.1 Master-Slave replication

Also called **primary-replica** or **leader-follower** replication.

One node accepts writes. Other nodes copy from it.

```text
            writes
App ───────────────→ Master DB
                       ↓
                       ↓ replication
                       ↓
               ┌───────┴────────┐
               ↓                ↓
           Slave DB 1        Slave DB 2

            reads             reads
```

In the food delivery app:

```text
Create order → Master Order DB
View order history → Slave Order DB
Track order status → Slave Order DB
```

The master handles writes like:

```text
Order placed
Payment authorized
Delivery assigned
Order canceled
```

The replicas handle reads like:

```text
Show my past orders
Show current order status
Load restaurant menu
```

This improves scalability because read traffic can be spread across replicas.

The weakness is that replicas may lag behind the master. A user might place an order, then immediately refresh and not see it for a second because the read went to a slightly stale replica.

This is called **replication lag**.

A common fix is **read-your-writes consistency**: after a user performs a write, send their immediate reads to the master or to a replica that has caught up.

---

## 2.2 Tree replication

**Tree replication** means data is copied through a hierarchy instead of one master sending updates directly to every replica.

```text
                Master
              /        \
        Replica A      Replica B
        /      \        /      \
     A1        A2     B1        B2
```

Why do this?

Because if the master has to replicate to thousands of replicas directly, it can become overloaded.

In a food delivery system, imagine restaurant menus are distributed globally.

```text
Menu Service Master
   ↓
Regional Replicas
   ↓
City-level Replicas
   ↓
Edge caches near users
```

Example:

```text
Global Menu DB
   ├── North America Menu Replica
   │      ├── New York Cache
   │      └── San Francisco Cache
   └── Europe Menu Replica
          ├── London Cache
          └── Paris Cache
```

This works well for data that is read heavily and updated less frequently, such as:

```text
restaurant menus
restaurant photos
delivery zones
promotions
product catalogs
configuration data
```

The benefit is efficient distribution at scale.

The downside is slower propagation. An update may need to travel through several levels before every cache has it.

So if a restaurant changes the price of a burger, some users may briefly see the old price until the update reaches their local cache.

---

## 2.3 Master-Master replication

**Master-master replication** means multiple nodes can accept writes.

```text
              writes
App ───────→ Master A
              ↕
              ↕ replication
              ↕
App ───────→ Master B
              writes
```

In a food delivery app, imagine two active regions:

```text
US-East Order DB  ←→  US-West Order DB
```

Users on the East Coast write to US-East. Users on the West Coast write to US-West.

This improves availability and latency.

If US-East fails, US-West can still accept orders. If a user is in California, their write does not need to travel all the way to Virginia.

But master-master replication introduces a hard problem: **conflicts**.

Example:

```text
Restaurant has 1 item left: "Sushi Combo"
User A orders it through US-East
User B orders it through US-West
Both regions accept the order at nearly the same time
```

Now both masters think they sold the last item.

This is a write conflict.

Master-master works best when conflicts are rare or easy to resolve. For example:

```text
user profile updates
shopping cart changes
analytics events
restaurant browsing history
```

It is harder for strongly consistent workflows like:

```text
payments
inventory
seat booking
bank transfers
order finalization
```

For those, systems often use a single writer, distributed locking, consensus, or partition ownership.

---

## 2.4 Buddy replication

**Buddy replication** means each node has a specific partner, or “buddy,” that stores a copy of its data.

```text
Node A ↔ Node B
Node C ↔ Node D
Node E ↔ Node F
```

If Node A fails, Node B has its data and can take over.

In a food delivery app, imagine a **delivery dispatch service** that manages active drivers in different city zones.

```text
Dispatch Node A: Manhattan orders
Dispatch Node B: Buddy backup for A

Dispatch Node C: Brooklyn orders
Dispatch Node D: Buddy backup for C
```

If Node A crashes while coordinating Manhattan deliveries, Node B can recover its state:

```text
active orders
assigned drivers
driver locations
delivery ETAs
```

Buddy replication is common in systems that partition work across nodes and want a targeted backup for each partition.

It is more efficient than replicating everything everywhere.

Instead of this:

```text
Every node stores every partition
```

You get this:

```text
Each node stores its own data plus a buddy’s data
```

The tradeoff is that if both a node and its buddy fail, that partition may become unavailable unless there is another backup.

---

# Putting it all together

A real food delivery architecture might use all of these patterns at once.

```text
Users
  ↓
Load Balancer
  ↓
API Gateway
  ↓
Microservices
  ↓
Databases, Caches, Queues
```

Availability patterns could look like this:

```text
API Gateway:
- active-active across regions
- fail-over through load balancer

Order Database:
- primary-replica replication
- automatic fail-over to a standby

Restaurant Menu Data:
- tree replication to regional caches and edge caches

User Profile Service:
- master-master replication across regions

Delivery Dispatch:
- buddy replication between partition owners

Message Queue:
- replicated brokers so order events are not lost
```

A simplified architecture:

```text
                         Users
                           ↓
                    Global Load Balancer
                    /                  \
             US-East Region        US-West Region
                  ↓                     ↓
             API Gateway           API Gateway
                  ↓                     ↓
            Order Service  ←────→  Order Service
                  ↓                     ↓
            Primary DB          Replica / Failover DB

            Menu Cache Tree:
            Global Menu DB
                 ↓
            Regional Cache
                 ↓
            City Cache

            Dispatch Nodes:
            Node A ↔ Node B
            Node C ↔ Node D
```

---

## Quick comparison

| Pattern           | Main idea                            | Best for                      | Main risk                       |
| ----------------- | ------------------------------------ | ----------------------------- | ------------------------------- |
| Fail-over         | Backup takes over when primary fails | Surviving crashes             | Fail-over delay, data loss risk |
| Replication       | Keep multiple copies                 | Availability and faster reads | Consistency problems            |
| Master-Slave      | One writer, many readers             | Read-heavy systems            | Replica lag                     |
| Tree replication  | Copy data through hierarchy          | Large-scale distribution      | Slower propagation              |
| Master-Master     | Multiple writers                     | Multi-region availability     | Write conflicts                 |
| Buddy replication | Each node has a paired backup        | Partitioned systems           | Buddy pair failure              |

---

## The mental model

Use this simple framing:

**Fail-over answers:**
“What happens when this thing dies?”

**Replication answers:**
“Where else does this data or service exist?”

**Master-slave answers:**
“Who is allowed to write, and who only copies?”

**Tree replication answers:**
“How do we distribute data to many places efficiently?”

**Master-master answers:**
“Can multiple places accept writes at the same time?”

**Buddy replication answers:**
“Who is responsible for backing up this specific node or partition?”





## 1. In an enterprise setting, how are two databases across two regions kept in sync?

Usually with **replication**.

In most enterprise systems, you do **not** usually run two fully equal writable databases in two regions unless you really need it. The common setup is:

```text
Primary Region                         Secondary Region
--------------                         ----------------
App writes here  ────────────────→     Replica DB
Primary DB                              Read-only / standby DB
```

The primary database accepts writes. The secondary database continuously receives changes from the primary.

For PostgreSQL, the rough idea is:

```text
Transaction happens on primary DB
        ↓
Postgres writes change to WAL
        ↓
WAL changes are streamed/copied to replica
        ↓
Replica replays those changes
        ↓
Replica becomes nearly up to date
```

**WAL** means **Write-Ahead Log**. It is PostgreSQL’s durable record of changes. Replication works by shipping those changes to another database and replaying them there.

So if your app writes:

```sql
INSERT INTO orders (id, user_id, restaurant_id, status)
VALUES (123, 45, 9, 'PLACED');
```

The primary database commits the transaction, records it in its WAL, and the secondary database receives and replays that change.

---

## The important distinction: synchronous vs asynchronous replication

### Synchronous replication

The primary does not consider a write successful until at least one replica confirms it received the change.

```text
App → Primary DB → Replica confirms → App gets success
```

This gives stronger durability, but it adds latency. Across regions, that latency can be significant because every write has to wait for a network round trip between regions.

This is more common **within a region**, across Availability Zones.

### Asynchronous replication

The primary confirms the write immediately, and the replica catches up shortly after.

```text
App → Primary DB → App gets success
              ↓
        Replica catches up later
```

This is the common model for **cross-region database replication**.

The downside is **replication lag**. The secondary region may be seconds, milliseconds, or sometimes more behind the primary. If the primary region fails before the latest changes reach the secondary, you can lose the last few committed writes.

That possible loss is called **RPO**, or **Recovery Point Objective**.

---

## Enterprise fail-over example

Imagine your production database is in `us-east-1`, and you maintain a replica in `us-west-2`.

```text
Normal state:

Users
  ↓
Global Load Balancer
  ↓
App in us-east-1
  ↓
Primary Postgres DB in us-east-1
  ↓ async replication
Read Replica / Standby DB in us-west-2
```

If `us-east-1` fails:

```text
Failure state:

Users
  ↓
Global Load Balancer
  ↓
App in us-west-2
  ↓
Promoted Postgres DB in us-west-2
```

The secondary database is **promoted** from read-only replica to writable primary.

After that, applications need to point writes to the new primary. In real enterprises, this usually involves:

```text
health checks
DNS or global traffic manager changes
database promotion
application fail-over
connection string / secret updates
runbooks or automation
post-failover validation
```

The hard part is not just “copy the data.” The hard part is coordinating traffic, writes, monitoring, fail-over, and recovery safely.

---

# 2. Using AWS hosted PostgreSQL, how can you leverage AWS for this?

On AWS, you have a few main choices depending on what you are using.

## Option A: Amazon RDS for PostgreSQL with cross-region read replica

If you are using **standard Amazon RDS for PostgreSQL**, the usual cross-region pattern is:

```text
RDS PostgreSQL Primary
Region A
   ↓
Cross-Region Read Replica
Region B
```

AWS supports creating cross-region read replicas for RDS PostgreSQL. AWS describes cross-region read replicas as useful for disaster recovery, read scaling, and cross-region migration. ([Amazon Web Services, Inc.][1])

Architecture:

```text
us-east-1
--------
Application
   ↓
RDS PostgreSQL Primary
   ↓ async replication

us-west-2
--------
RDS PostgreSQL Read Replica
```

In normal operation:

```text
Writes → primary region
Reads  → primary region, or sometimes local read replica
```

During disaster recovery:

```text
1. Primary region fails
2. Promote the cross-region read replica
3. Make the promoted DB writable
4. Point the application in the secondary region to the new DB
5. Route user traffic to the secondary region
```

AWS lets you create cross-region read replicas through the RDS console, CLI, or API. Under the hood, RDS starts with a snapshot/copy process and then continues replication from the source DB to the replica. ([AWS Documentation][2])

This is a good fit when you want:

```text
managed PostgreSQL
cross-region disaster recovery
mostly single-region writes
lower operational burden
```

But remember: this is generally **asynchronous** cross-region replication. The replica can lag, so your disaster recovery plan must define acceptable RPO and RTO.

---

## Option B: Amazon Aurora PostgreSQL Global Database

If you can use **Amazon Aurora PostgreSQL-compatible** instead of standard RDS PostgreSQL, AWS has a more specialized multi-region option: **Aurora Global Database**.

Architecture:

```text
Primary AWS Region
------------------
Aurora PostgreSQL writer cluster
        ↓
Aurora Global Database replication
        ↓
Secondary AWS Region
--------------------
Aurora PostgreSQL read-only cluster
```

AWS says Aurora Global Database is designed for globally distributed applications, low-latency global reads, and recovery from region-wide outages. A global database consists of Aurora clusters in multiple AWS Regions. ([AWS Documentation][3])

This is often the better AWS-native answer for enterprise-grade multi-region PostgreSQL-compatible disaster recovery.

Normal state:

```text
Users in US-East → app in us-east-1 → Aurora writer in us-east-1
Users in US-West → app in us-west-2 → Aurora reader in us-west-2 for reads
```

Fail-over state:

```text
us-east-1 fails
   ↓
Promote secondary Aurora cluster in us-west-2
   ↓
Applications send writes to us-west-2
```

AWS provides managed switchover/failover workflows for Aurora Global Database to support business continuity and disaster recovery. ([AWS Documentation][4])

Aurora Global Database is commonly preferred when you need:

```text
lower RPO/RTO than basic cross-region replicas
multi-region read locality
managed regional disaster recovery
PostgreSQL compatibility
```

The catch: Aurora PostgreSQL is compatible with PostgreSQL, but it is not exactly the same product as standard RDS PostgreSQL. You need to check extension support, version support, cost, operational constraints, and application compatibility.

---

## Option C: Multi-AZ RDS PostgreSQL

This is important but often confused with cross-region replication.

**Multi-AZ is for high availability inside one region**, not across two regions.

```text
AWS Region: us-east-1

Availability Zone A → writer
Availability Zone B → standby/reader
Availability Zone C → reader
```

AWS RDS Multi-AZ DB clusters place a writer and two readable replicas in three separate Availability Zones within the same Region. ([AWS Documentation][5])

Use Multi-AZ for:

```text
server failure
storage failure
AZ failure
maintenance fail-over
regional high availability
```

Use cross-region replicas or Aurora Global Database for:

```text
region-wide disaster recovery
multi-region availability
regional evacuation
```

A strong enterprise setup often combines both:

```text
Primary Region:
RDS/Aurora with Multi-AZ

Secondary Region:
Cross-region replica or Aurora Global Database secondary
```

---

# Recommended AWS architecture for your case

For a serious enterprise system using AWS hosted PostgreSQL, I would think in two tiers.

## Baseline production architecture

```text
Users
  ↓
Route 53 / Global Accelerator
  ↓
Application Load Balancer
  ↓
App services in primary region
  ↓
RDS PostgreSQL Multi-AZ
  ↓
Cross-region read replica
```

Example:

```text
us-east-1
--------
App Service
RDS PostgreSQL Multi-AZ Primary

        ↓ async cross-region replication

us-west-2
--------
Warm standby App Service
RDS PostgreSQL Cross-Region Read Replica
```

Failure plan:

```text
1. Detect primary region failure
2. Stop or fence writes to old primary if possible
3. Promote read replica in secondary region
4. Update DB endpoint/config/secrets
5. Shift traffic using Route 53 or Global Accelerator
6. Validate app health
7. Rebuild replication in the opposite direction later
```

This gives you a **warm standby** disaster recovery model.

---

## Better enterprise architecture if Aurora is acceptable

```text
Users
  ↓
Route 53 / Global Accelerator
  ↓
App in multiple regions
  ↓
Aurora PostgreSQL Global Database
```

```text
us-east-1
--------
Aurora primary writer cluster

        ↓ managed global replication

us-west-2
--------
Aurora secondary read-only cluster
```

Normal operation:

```text
Writes → primary region
Reads  → local regional readers where safe
```

During outage:

```text
Promote secondary region
Route traffic to secondary app stack
Resume writes there
```

This is more purpose-built for cross-region PostgreSQL-compatible availability than basic RDS cross-region replicas.

---

# Important design warning

Cross-region PostgreSQL replication usually does **not** mean both databases should accept writes at the same time.

This is tempting:

```text
us-east-1 DB accepts writes
us-west-2 DB accepts writes
```

But it creates conflict problems:

```text
Same order updated in both regions
Same inventory item sold twice
Same payment processed twice
Conflicting user profile changes
```

For most enterprise systems, the safer pattern is:

```text
single-writer, multi-reader
```

That means:

```text
one writable primary region
one or more read-only secondary regions
controlled promotion during fail-over
```

For critical workflows like orders, payments, inventory, reservations, and balances, single-writer is much easier to reason about.

---

## Simple answer

For AWS hosted PostgreSQL:

| Need                                                 | AWS feature                                                             |
| ---------------------------------------------------- | ----------------------------------------------------------------------- |
| Survive instance/AZ failure                          | RDS Multi-AZ                                                            |
| Have a standby DB in another region                  | RDS PostgreSQL cross-region read replica                                |
| Faster multi-region DR with PostgreSQL compatibility | Aurora PostgreSQL Global Database                                       |
| Read locally from other regions                      | Cross-region read replicas or Aurora Global Database secondary clusters |
| Write in multiple regions at the same time           | Avoid unless you have a very deliberate conflict-resolution design      |

For the food-delivery example: I would use **single-region writes** for orders and payments, **Multi-AZ** inside the primary region, and either **RDS cross-region read replica** or **Aurora Global Database** for disaster recovery.

[1]: https://aws.amazon.com/blogs/database/best-practices-for-amazon-rds-for-postgresql-cross-region-read-replicas/?utm_source=chatgpt.com "Best practices for Amazon RDS for PostgreSQL cross-Region read replicas ..."
[2]: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.XRgn.html?utm_source=chatgpt.com "Creating a read replica in a different AWS Region"
[3]: https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html?utm_source=chatgpt.com "Using Amazon Aurora Global Database - Amazon Aurora"
[4]: https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database-disaster-recovery.html?utm_source=chatgpt.com "Using switchover or failover in Amazon Aurora Global Database"
[5]: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/multi-az-db-clusters-concepts.html?utm_source=chatgpt.com "Multi-AZ DB cluster deployments for Amazon RDS"
