# System Design Content Updates Roadmap

Reference doc comparing this folder to the [system-design-primer](https://github.com/donnemartin/system-design-primer) **core topic index** (excluding "Design X like Y" interview exercises). Use this when deciding what to add or extend next.

**Last reviewed:** 2026-05-22  
**Active guides:** 11 numbered files in this folder (see numbering below).

---

## What you already cover well

| Area | Guides |
|------|--------|
| CAP / PACELC / consistency spectrum | `2.1`, `2.2` |
| Availability, failover, replication | `3`, `1` (scaling section) |
| RDBMS, NoSQL, ACID, BASE, store types | `5` |
| Caching (layers + read/write patterns) | `4` |
| Sharding / partitioning / object storage | `1` |
| Async, EDA, CQRS, event sourcing | `7` |
| Kafka vs messaging / task vs log | `8` |
| Stability (timeout, breaker, bulkhead, throttle) | `8.stability-patterns` |
| Concurrency models | `6` |
| Traffic estimation (back-of-envelope) | `10` |
| GraphQL / federation / BFF | `9` |
| Real-world edge/CDN walkthrough | `11` |
| Request path (OSI, L4/L7, DDoS, VIP/Anycast) | `1` |

**Beyond primer:** PACELC, stability patterns, deep EDA/Kafka, GraphQL, traffic estimation, intuit.com CDN example.

---

## Recommended content updates

Priority is for **interview frequency** and **primer “start here”** topics. Each item suggests *new guide* vs *extend existing*.

### High priority

| # | Topic | Gap | Suggested action | Target |
|---|--------|-----|------------------|--------|
| 1 | **Performance vs scalability** | No crisp “one user slow” vs “slow under load” framing | New short guide **or** new § in `1` | `12.performance-scalability-study-guide.md` or `1` §5 |
| 2 | **Latency vs throughput** | Mentioned but not standalone mental model | Same guide as #1 (pair topics) | Same as #1 |
| 3 | **Availability in numbers** | No nines table, downtime budgets, serial/parallel availability math | New guide or § in `3` | `3` new section or `12` |
| 4 | **DNS deep dive** | DNS is one hop in flow; missing record types, TTL, routing policies | New guide or expand `11` + link from `1` | `12.dns-study-guide.md` or `11` + `1` |
| 5 | **CDN push vs pull** | ✓ Done — added as §10 in `11` | — | `11` §10 |
| 6 | **Interview approach (4 steps)** | Primer: use cases → HLD → components → scale; only “clarify” in `10` | New meta guide | `0.system-design-interview-approach.md` or appendix in `10` |
| 7 | **Latency numbers cheat sheet** | No L1/RAM/SSD/network/DC RTT anchors | One-page appendix | `10` appendix or `13.latency-numbers-reference.md` |

### Medium priority

| # | Topic | Gap | Suggested action | Target |
|---|--------|-----|------------------|--------|
| 8 | **Load balancer / reverse proxy** | L4/L7 in `1`; missing LB vs reverse proxy, sticky sessions, health checks as interview bullets | Dedicated § or small guide | Extend `1` or `12.load-balancing-study-guide.md` |
| 9 | **Security fundamentals** | DDoS + GraphQL abuse only; no authn/authz, TLS, secrets, input validation overview | New guide | `12.security-study-guide.md` |
| 10 | **Communication: TCP, RPC, REST** | UDP in `1`; gRPC/REST mentions in `9`; no sync-protocol comparison | New guide | `12.communication-protocols-study-guide.md` |
| 11 | **Service discovery** | etcd/ZK only as CP stores in `2.2`; no client→service registry patterns | New § in microservices topic or guide | Pair with #10 or `9` |

### Lower priority (often folded into DB/architecture answers)

| # | Topic | Gap | Suggested action | Target |
|---|--------|-----|------------------|--------|
| 12 | **Database federation** | Split DBs by bounded context (not GraphQL federation) | § in `5` | `5` new section |
| 13 | **SQL tuning** | Indexes mentioned; no query plans, N+1 at SQL layer, connection pooling | § in `5` | `5` new section |
| 14 | **Weak consistency (primer label)** | Eventual/causal covered; VoIP/memcached “weak” not named | One subsection | `2.2` consistency levels |
| 15 | **Task queues vs message queues** | Strong in `8`; could cross-link primer “asynchronism” explicitly | Cross-links only | `7`, `8`, `1` |

---

## Partial coverage — extend, don’t duplicate

| Primer topic | You have | Extend with |
|--------------|----------|-------------|
| Consistency patterns | `2.2` levels + `5` BASE | Add explicit **weak consistency** row (VoIP, realtime) |
| Back pressure | `6`, `8` | Link from `7` asynchronism section |
| Microservices | `9` federation/BFF | Service boundaries, discovery, deploy independence |
| Powers of two | `10` constants table | Optional link to dedicated latency/bytes appendix |

---

## Suggested new file numbering (if you add guides)

Keep existing `1`–`11`. Options:

- **`0.*`** — Interview process / how to run the interview (meta)
- **`12.*`** — Fundamentals bundle (perf/latency, DNS, LB, security, protocols) — split or one “infrastructure primitives” guide
- **`13.*`** — Latency numbers reference only (thin cheat sheet)

Or **avoid new files** by bolting high-priority items onto:

- `1` — perf/scalability, latency/throughput, LB/reverse proxy depth
- `3` — availability nines + serial/parallel math
- `5` — SQL tuning + database federation
- `10` — interview 4-step framework + latency numbers appendix
- `11` — DNS records + CDN push vs pull

---

## Primer topics explicitly out of scope (per your request)

Do **not** add guides for these unless you change scope:

- Design Pastebin, Twitter, web crawler, Mint, social graph, etc.
- Object-oriented design exercises (hash map, LRU, parking lot, …)
- Curated lists of company engineering blogs / external architecture links (optional reading list only)

---

## Quick gap checklist

Use when reviewing a new draft:

- [ ] Performance vs scalability defined in one sentence each
- [ ] Latency vs throughput with “maximize throughput, acceptable latency”
- [ ] 99.9% / 99.99% downtime table + serial vs parallel availability formula
- [ ] DNS: A, CNAME, MX, NS, TTL, weighted/latency/geo routing
- [x] CDN: push vs pull tradeoffs
- [ ] LB vs reverse proxy; sticky sessions; health checks
- [ ] Service discovery (K8s DNS, Consul, mesh)
- [ ] Security: authn/authz, TLS, rate limits, validation (beyond DDoS)
- [ ] TCP vs UDP vs RPC vs REST when to pick what
- [ ] Database federation vs sharding vs GraphQL federation (disambiguate)
- [ ] SQL tuning: indexes, explain, pooling, avoid N+1
- [ ] Latency numbers: L1, RAM, SSD, same-DC, cross-region
- [ ] Interview: 4-step framework linked to `10` estimation

---

## Cross-reference: primer index → your guides

| Primer index section | Primary guide(s) | Status |
|----------------------|------------------|--------|
| Performance vs scalability | — | **Missing** |
| Latency vs throughput | `10`, scattered | **Partial** |
| CAP / consistency / availability patterns | `2.1`, `2.2`, `3`, `5` | **Strong** |
| Availability in numbers | — | **Missing** |
| DNS | `1`, `11` | **Partial** |
| CDN | `4`, `11` | **Strong** |
| Load balancer / reverse proxy | `1`, `3` | **Partial** |
| Microservices / service discovery | `9` | **Partial** / discovery **missing** |
| Database (RDBMS, NoSQL, replication, sharding) | `1`, `5` | **Strong** |
| Database federation, SQL tuning | — | **Missing** |
| Cache | `4` | **Strong** |
| Asynchronism | `1`, `6`, `7`, `8` | **Strong** |
| Communication (TCP, UDP, RPC, REST) | `1`, `9` | **Partial** |
| Security | `1` (DDoS), `9` (GraphQL) | **Weak** |
| Appendix (powers of two, latency numbers) | `10` | **Partial** |
| How to approach interview | `10` clarify only | **Partial** |

---

## Notes

- `archive/` is treated as superseded; do not expand unless consolidating history.
- After adding content, update each guide’s **Table of Contents** and cross-links at the top (same pattern as `2.2` → `3`, `5`).
- Prefer **30–45 minute study guide** format already used in this folder for consistency.
