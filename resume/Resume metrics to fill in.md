# Resume metrics worksheet

Use this document to collect **real numbers** (or defensible ranges) for placeholders in `Last Resume - revised.md`.  
Search: **`[`** in the revised resume to find every slot.

**Tips**

- Ranges are fine: “~40%”, “low double-digit %”, “millions of events/day”.
- If exact numbers are confidential, use **relative** improvement or **order of magnitude**.
- One strong metric per bullet beats three vague ones.
- Delete the placeholder entirely if you truly have no number — tighten the bullet instead.

---

## Intuit (highest priority)

### Fulfillment / inventory sync subsystem

| Placeholder | What to find | Where to look / how to estimate |
|-------------|--------------|----------------------------------|
| **[N] sales channels** | Count of channel integrations (Shopify, Amazon, etc.) | Team docs, architecture diagram, on-call runbooks |
| **Fault tolerance story** | Retries, DLQ rate, SLO uptime, incident count before/after | Splunk/Datadog, incident tickets, SLO dashboards |
| **Ownership scope** | “Owned” vs “contributed” — team size, on-call | Self-review, RFC author, primary on-call rotation |

**Questions to answer**

1. How many sales channels (or connector types) does the sync support?
2. What is approximate **daily/hourly event volume** on the inventory topic?
3. Any **SLO** (e.g. 99.9% successful sync within X minutes)?
4. Did you reduce **incidents** or **manual interventions**? By how much?

---

### Throughput / near-real-time sync bullet

| Placeholder | What to find | Where to look |
|-------------|--------------|---------------|
| **[X] to [Y] TPS** | Messages or records consumed per second before vs after your change | Metrics dashboard, load test results, PR description |
| **~Z%** | Alternative if you only know percentage improvement | Before/after dashboard screenshot dates |
| **[A] to [B] lag/backlog** | Consumer lag, queue depth, or “time until synced” | Kafka consumer lag, queue depth alerts |
| **Technique** | Partitioning, more consumers, batching, DB tuning, etc. | Your PRs, design doc, postmortem |

**Questions to answer**

1. Throughput before your work: _____ TPS (or msgs/min).
2. Throughput after: _____ TPS (or **~___%** increase).
3. p99 latency or **end-to-end sync delay** before/after: _____ → _____.
4. Peak traffic scenario (Black Friday, month-end): what broke before, what holds now?

---

### Shipping module

| Placeholder | What to find |
|-------------|--------------|
| **Carriers / APIs** | Names you’re allowed to put on resume (USPS, UPS, internal Ship API, etc.) |
| **Users / volume** | # of shipments/month, % of QBO customers using shipping, GMV — pick one |
| **Scope** | Led design? # of engineers? API count? |

**Questions to answer**

1. Did you **lead** design or **implement** under another lead?
2. Rough **adoption**: users, invoices shipped/month, or “launched to GA for ___”.
3. Any **latency/reliability** metric (label print time, API success rate)?

---

### Credential store

| Placeholder | What to find |
|-------------|--------------|
| **Rotation** | Rotation frequency (e.g. every 90 days), automated vs manual |
| **Scale** | # credentials/secrets, # services onboarded |
| **Impact** | Manual tickets eliminated, audit findings closed, time saved |

**Questions to answer**

1. How many **secrets/credentials** does the store manage (order of magnitude)?
2. Was rotation **fully automated**? What failed before this existed?
3. Security/compliance win (passed audit, reduced vault incidents)?

---

## TriNet

### Authorization service

| Placeholder | What to find |
|-------------|--------------|
| **~X% latency** | p50/p95/p99 before vs after caching + SQL changes |
| **Scale** | Requests/sec, # services using authZ, # ACL rules |
| **Dashboard** | # policy changes per month without deploy |

**Questions to answer**

1. p99 authorization latency: _____ ms → _____ ms.
2. Cache hit rate after Redis: _____%.
3. How many **microservices** depended on this service?

---

### SSO self-service

| Placeholder | What to find |
|-------------|--------------|
| **~X% support reduction** | Tickets/month before vs after (SSO / password category) |
| **Adoption** | # clients onboarded via self-service vs manual |

**Questions to answer**

1. Forgot-password or SSO setup tickets: _____/month → _____/month.
2. Time to onboard a new SAML client: _____ days → _____ days.

---

### Messaging framework + dashboard

| Placeholder | What to find |
|-------------|--------------|
| **Volume** | Messages/day across the bus |
| **Recovery** | % failed messages resubmitted successfully; MTTR for stuck messages |

**Questions to answer**

1. Average **messages/day** (or peak)?
2. **Audit** requirement — what compliance need did this solve?
3. Ops time saved per incident using resubmit dashboard?

---

## Optional additions (only if true)

Add a short **Leadership** or **Impact** bullet at Intuit if applicable:

| Fact | Example bullet line |
|------|---------------------|
| Mentored engineers | Mentored **[N]** engineers on **[area]** |
| Tech lead | Technical lead for **[team/feature]** (**[N]** engineers) |
| On-call | Primary on-call for **[subsystem]**; reduced MTTR from **[A]** to **[B]** |
| Design docs | Authored RFCs for **[feature]** adopted by **[N]** teams |
| Incidents | Led postmortem for **[incident]**; shipped guards preventing recurrence |

---

## Placeholder → filled text (copy when done)

Fill in below, then replace brackets in `Last Resume - revised.md`.

```
Intuit — channels:        [N] = _______________
Intuit — TPS before:      [X] = _______________
Intuit — TPS after:       [Y] = _______________
Intuit — or % gain:       [Z] = _______________
Intuit — lag before:      [A] = _______________
Intuit — lag after:       [B] = _______________
Intuit — tuning note:     _______________

TriNet — latency %:       [X] = _______________
TriNet — SSO ticket drop: [X] = _______________
TriNet — messaging vol:   _______________
```

---

## Redaction check before sending

- [ ] No unreleased product names if policy forbids it  
- [ ] No customer PII or internal-only project codenames  
- [ ] Metrics approved for external use (some companies restrict “exact” production numbers)  
- [ ] Export **PDF** from this markdown for applications (plain header, no embedded images)
