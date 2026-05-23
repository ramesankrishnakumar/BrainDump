Krishnakumar Ramesan

Hayward, CA | (256) 652-8738 | [linkedin.com/in/rkrrish](https://www.linkedin.com/in/rkrrish/) | [github.com/ramesankrishnakumar](https://github.com/ramesankrishnakumar)

**Senior Software Engineer | Distributed Systems | SSO & Event-Driven Architecture**

---

## Profile

Senior Software Engineer with **13+ years** building enterprise microservices, identity integrations (**SAML, OIDC, Okta**), and event-driven systems (**Kafka, RabbitMQ**). At Intuit, own commerce fulfillment and near-real-time inventory sync across QuickBooks sales channels. Strong in **Kotlin/Java**, API design, security (envelope encryption, KMS, mTLS), and production reliability (idempotency, retries, HPA-based autoscaling).

---

## Skills

**Languages:** Kotlin, Java, Python  
**Frameworks:** Spring Boot, Hibernate, GraphQL (Netflix DGS)  
**Messaging / streaming:** Kafka, RabbitMQ  
**Data:** PostgreSQL, DynamoDB, Redis  
**Identity / security:** Okta, SAML, OIDC, SSO, AWS KMS, AES-256-GCM  
**Platform:** AWS, Kubernetes, Podman  

---

## Experience

### Intuit — Senior Software Engineer | Aug 2022 – Present

Commerce platform for QuickBooks; fulfillment, inventory sync, shipping, and secure credentials.

- **Owned** a fault-tolerant fulfillment subsystem syncing inventory changes across **4 sales channels** (hundreds of thousands of events/day); designed for idempotent consumers, ordered delivery, and safe recovery under partial failures.
- **Validated 75 TPS** throughput for GA (against a 70 TPS traffic-estimate baseline) using custom datagen scripts; confirmed published-to-consumed event accuracy end-to-end across the product → variant → fulfillment dependency chain.
- **Configured Kubernetes HPA** driven by Kafka consumer lag to auto-scale pods under variable load; combined with realm_id sharding across 2 shards to sustain throughput targets.
- **Diagnosed and resolved** a 30-second API timeout in catalog variant bulk-matching: traced root cause via Hibernate query logs (top queries by count and p90 latency) and `EXPLAIN ANALYZE`, eliminated N+1 loads, overfetching, and sequential scans — **response time dropped from 30 s to 785 ms (~97%)**.
- **Designed and built** account and credential management for QuickBooks' shipping module — enabling customers to fulfill invoices end-to-end via a carrier aggregator integration; launched to GA as the **first core revenue product in the Commerce Business Unit**, with hundreds of shipping labels created post-launch.
- **Designed and built** a credential store managing **thousands** of shipping API credentials (hierarchical parent/child keys, one child per connected company); implemented **AES-256-GCM envelope encryption** via AWS KMS — root key never leaves the keystore, rotated every **30 days**, with key-version headers on all encrypted records for seamless rotation. Data secured at rest (KMS) and in transit (**mTLS** over service mesh). Architected as an isolated module with a clean migration path to the platform credential system.

**Stack (typical):** Kotlin, Java, Kafka, PostgreSQL, DynamoDB, AWS, Kubernetes

---

### TriNet — Software Engineer | Aug 2016 – Jul 2022

Authorization, SSO platform, and internal messaging for HR/product microservices.

- **Built** a scalable ACL-based authorization service consumed by **20 microservices**; eliminated per-request DB roundtrips via two-tier Redis caching — employee roles cached by `companyId:employeeId` with event-driven eviction on role change, endpoint access policies at a longer TTL — significantly reducing p90 authorization latency under production load.
- **Delivered** a companion dynamic policy dashboard allowing DevOps to update endpoint role requirements and toggle access without code redeploys; decoupled access policy changes from the deployment cycle across 20 dependent services.
- **Built** a core SSO microservice serving **~400K TriNet customers** across **~20 cloud software provider integrations**; resolves provider-specific SAML attribute values not available in the IAM, surfaces per-user provisioned application links, and orchestrates the final SSO handshake — enabling password-free access for employees at small/mid-size companies with no dedicated IT resources.
- **Developed** a SAML integration test stub enabling QE to run full SSO regression suites without live partner test environments — decoupled release testing from third-party availability.
- **Built** an audited event-bridge connecting an on-premise HR engine to new microservices during a phased decomposition; processes **300–400K events/day** with write-ahead audit logging (replay buffer when the messaging system is unreachable); delivered an admin dashboard for support to query, correct, and resubmit failed events — **saving 10–15 min per incident** vs. error-prone manual SQL fixes — with a full change audit trail capturing every human intervention.

**Stack (typical):** Java, Spring, Redis, PostgreSQL, RabbitMQ, SAML/OIDC

---

### Earlier experience

**Mazda North American Operations** — Developer Contractor | Nov 2015 – Apr 2016  
RESTful microservices during monolith modernization; performance and feature work on internal apps.

**Infosys Limited** — Systems Engineer | Jun 2011 – Jun 2013  
ETL job monitoring/support; migration/testing in lower environments; daily job statistics reporting.

---

## Education

**M.S. Computer Science**, University of Alabama in Huntsville — May 2015 (GPA **3.75 / 4.0**)  
**B.Tech Electronics and Communication**, Amrita School of Engineering — Jun 2011 (GPA **3.0 / 4.0**)
