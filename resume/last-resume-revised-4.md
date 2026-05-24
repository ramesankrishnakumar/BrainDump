Krishnakumar Ramesan

Hayward, CA | (256) 652-8738 | r.krishnakumar90@gmail.com | [linkedin.com/in/rkrrish](https://www.linkedin.com/in/rkrrish/) | [github.com/ramesankrishnakumar](https://github.com/ramesankrishnakumar)

**Senior Software Engineer | Distributed Systems | Multi-Agent AI | SSO & Event-Driven Architecture**

---

## PROFILE

Senior Software Engineer with **13+ years** building enterprise microservices, identity integrations with **SAML, OIDC, and Okta**, and event-driven systems on **Kafka and RabbitMQ**. Recently designed and shipped a production multi-agent AI onboarding system for QuickBooks, ramped to **50% of first-time trial customers**. Strong in **Kotlin/Java**, distributed system design, data security, and production reliability at scale.

---

## EXPERIENCE

### Intuit — Senior Software Engineer | Aug 2022 – Present

- **Shipped** a production multi-agent AI onboarding assistant now serving **50% of first-time QuickBooks trial customers**, achieving ~50% module-completion with consistent multi-session engagement. Built on LangGraph with an intent orchestrator that routes each message to a journey orchestrator coordinating 7 domain sub-agents, a focused single-module agent, or a live expert handoff for overwhelmed customers. Drove from hackathon prototype to production rollout.
- **Built** the real-time screen context engine powering those agents, capturing live DOM state and combining it with conversation history, user progress, and profile so agents deliver precise navigational and onboarding instructions without requiring backend screen telemetry.
- **Owned** a fault-tolerant fulfillment subsystem syncing inventory changes across **4 sales channels** (hundreds of thousands of events/day), designed for idempotent consumers, ordered delivery, and safe recovery under partial failures. Validated **75 TPS** throughput for GA using custom load-generation scripts and configured Kubernetes HPA driven by Kafka consumer lag to sustain throughput without over-provisioning.
- **Eliminated** a production 30-second timeout in the catalog variant bulk-match flow by profiling query patterns, identifying redundant round-trips, unnecessary data loads, and missing indexes, **dropping response time from 30 s to 785 ms (97% reduction)**.
- **Designed and built** account and credential management for QuickBooks' shipping module, enabling customers to fulfill invoices end-to-end via a carrier aggregator. Launched to GA as the **first core revenue product in the Commerce Business Unit** with hundreds of shipping labels created post-launch. Secured **thousands of carrier API credentials** with AES-256-GCM envelope encryption via AWS KMS, 30-day root key rotation, and mTLS in transit.
- **Designed and built** a configurable peer-review system for Live Bookkeeping expert work with FIFO-based reviewer assignment, structured questionnaire with bidirectional notifications, and UiPath RPA integration for automated final-stage review. Zero-code extension to new services via configuration alone, **saving ~25,000 hours of expert time (30 min × 50K active services)**.

---

### TriNet — Software Engineer | Aug 2016 – Jul 2022

- **Built** a scalable ACL-based authorization service consumed by **20 microservices**, eliminating per-request database round-trips with a two-tier cache (per-user role entries invalidated on role-change events, endpoint access policies at a longer TTL), measurably reducing p90 authorization latency under production load. Delivered a companion dynamic policy dashboard allowing DevOps to update role requirements without code redeploys across all 20 dependent services.
- **Built** a core SSO microservice serving **~400K TriNet customers** across **~20 cloud software provider integrations**, resolving provider-specific SAML attributes, surfacing per-user provisioned application links, and orchestrating the final SSO handshake to enable password-free access for employees with no dedicated IT resources.
- **Eliminated** dependency on live partner environments for SSO regression testing by building a SAML integration test stub covering **~20 provider integrations**, enabling QE to run full regression suites on-demand and decoupling release cycles from third-party availability.
- **Built** an audited event-bridge connecting an on-premise HR engine to new microservices, processing **300–400K events/day** with write-ahead audit logging as a replay buffer for failure recovery. Delivered an admin dashboard for support to query, correct, and resubmit failed events, **saving 10–15 min per incident**, with a full change audit trail on every human intervention.

---

### Mazda North American Operations — Developer Contractor | Nov 2015 – Apr 2016

- Built RESTful microservices during a monolith modernization initiative; delivered performance improvements and feature enhancements to internal applications.

---

### Infosys Limited — Systems Engineer | Jun 2011 – Jun 2013

- Monitored and supported production ETL jobs, migrated and tested code across lower environments, and reported daily job statistics to business stakeholders.

---

## EDUCATION

**M.S. Computer Science**, University of Alabama in Huntsville — May 2015 | GPA **3.75 / 4.0**  
**B.Tech Electronics and Communication**, Amrita School of Engineering — Jun 2011 | GPA **3.0 / 4.0**

---

## SKILLS

**Languages:** Kotlin, Java, Python  
**Frameworks:** Spring Boot, Hibernate, GraphQL (Netflix DGS)  
**AI / Agents:** LangGraph, OpenAI API, Vision LLM, multi-agent orchestration  
**Messaging / Streaming:** Kafka, RabbitMQ  
**Data:** PostgreSQL, DynamoDB, Redis  
**Identity / Security:** Okta, SAML, OIDC, SSO, AWS KMS, AES-256-GCM  
**Platform:** AWS, Kubernetes, Podman
