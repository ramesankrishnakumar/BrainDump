Krishnakumar Ramesan

Hayward, CA | (256) 652-8738 | ramesankrishnakumar@gmail.com | [linkedin.com/in/rkrrish](https://www.linkedin.com/in/rkrrish/) | [github.com/ramesankrishnakumar](https://github.com/ramesankrishnakumar)

**Senior Software Engineer | Distributed Systems | Multi-Agent AI | SSO & Event-Driven Architecture**

---

## PROFILE

Senior Software Engineer with **13+ years** building enterprise microservices, identity integrations with **SAML, OIDC, and Okta**, and event-driven systems on **Kafka and RabbitMQ**. Recently designed and shipped a production multi-agent AI onboarding system for QuickBooks on **LangGraph and OpenAI GPT 4.1**, ramped to **50% of first-time trial customers**. Strong in **Kotlin/Java**, distributed system design, data security, and production reliability at scale.

---

## EXPERIENCE

### Intuit — Senior Software Engineer | Aug 2022 – Present

- **Designed and shipped** a production multi-agent AI onboarding assistant for QuickBooks on LangGraph and OpenAI GPT 4.1, driving it from hackathon prototype to production as tech lead. Built a config-driven intent orchestrator that classifies each user message as an onboarding question, an expert handoff, or a general QuickBooks question beyond the domain agents' scope, then routes onboarding questions to a second config-driven orchestrator that delegates across 7 domain agents or answers directly from prior conversation and progress context.
- **Built** the screen context engine grounding those agents in what the customer sees, converting the live DOM into a compact markdown structure with on-page redaction of sensitive data like SSNs, addresses, and phone numbers. Replaced an earlier screenshot-plus-vision-LLM pipeline of roughly **9 seconds per turn with a ~30 ms DOM parse**, trading away on-screen imagery the onboarding use case did not need.
- **Hardened** the context engine for a constantly navigating app by debouncing captures to fire only after the user settles, waiting for page navigation to complete before reading the DOM, and letting agents request a fresh read on demand, keeping guidance accurate without flooding the pipeline on every interaction.
- **Designed and built** account and credential management for QuickBooks' shipping module, enabling customers to fulfill invoices end to end and print labels via a carrier aggregator. Launched to GA as the **first core revenue product in the Commerce Business Unit** with hundreds of shipping labels created post-launch.
- **Secured** thousands of carrier API credentials with AES-256-GCM envelope encryption via AWS KMS, 30-day versioned root key rotation, and mTLS in transit, built as an isolated migration-ready module to unblock GA when the platform team could not support the provider's hierarchical key model.
- **Owned** a fault-tolerant fulfillment subsystem ingesting events from **4 sales channels** (hundreds of thousands of events per day), designed for idempotent consumers, ordered delivery, and safe recovery under partial failures. Fulfillment events are linked to upstream product and variant entities before being published for downstream inventory consumers, making ordering a core design constraint. Scaled the pipeline to sustain **75 TPS** through database partitioning, sharding, and load-based autoscaling, verifying throughput with custom load-generation scripts.
- **Eliminated** a production 30-second timeout when bulk-matching catalog variants by profiling query patterns and removing redundant round-trips, unnecessary data loads, and missing indexes, **dropping response time from 30 s to 785 ms (97% reduction)**.
- **Designed and built** a configurable peer-review system for Live Bookkeeping expert work with FIFO-based reviewer assignment, structured questionnaires with bidirectional notifications, and UiPath RPA integration for automated final-stage review. New services onboard through configuration alone, **saving ~25,000 hours of expert time (30 min × 50K active services)**.

---

### TriNet — Software Engineer | Aug 2016 – Jul 2022

- **Built** a scalable ACL-based authorization service consumed by **20 microservices**, eliminating per-request database round-trips with a two-tier cache (per-user role entries invalidated on role-change events, endpoint access policies at a longer TTL), measurably reducing p90 authorization latency under production load. Delivered a companion dynamic policy dashboard allowing DevOps to update role requirements without code redeploys across all 20 dependent services.
- **Built** a core SSO microservice serving **~400K TriNet customers** across **~20 cloud software provider integrations**, resolving provider-specific SAML attributes, surfacing per-user provisioned application links, and orchestrating the final SSO handshake to enable password-free access for employees with no dedicated IT resources.
- **Eliminated** dependency on live partner environments for SSO regression testing by building a SAML integration test stub covering **~20 provider integrations**, enabling QE to run full regression suites on-demand and decoupling release cycles from third-party availability.
- **Built** an audited event-bridge connecting an on-premise HR engine to new microservices, processing **300–400K events/day** with write-ahead audit logging as a replay buffer for failure recovery. Delivered an admin dashboard for support to query, correct, and resubmit failed events, **saving 10–15 min per incident**, with a full change audit trail on every human intervention.

---

### Mazda North American Operations — Contractor | Nov 2015 – Apr 2016

- Built RESTful microservices during a monolith modernization initiative, delivering performance improvements and feature enhancements to internal applications.

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
**AI / Agents:** LangGraph, OpenAI API (GPT 4.1), multi-agent orchestration, DOM-based context engineering  
**Messaging / Streaming:** Kafka, RabbitMQ  
**Data:** PostgreSQL, DynamoDB, Redis  
**Identity / Security:** Okta, SAML, OIDC, SSO, AWS KMS, AES-256-GCM  
**Platform:** AWS, Kubernetes, Podman
