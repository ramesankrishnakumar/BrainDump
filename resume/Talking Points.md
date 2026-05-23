# Resume Talking Points
_Use this when an interviewer asks you to "tell me more" about a bullet. Raw detail so you don't have to reconstruct it on the spot._

---

## Intuit — Senior Software Engineer (Aug 2022 – Present)

### Fulfillment / Inventory Sync Subsystem

**Resume bullet:** Owned fault-tolerant fulfillment subsystem syncing inventory across 4 sales channels (hundreds of thousands of events/day).

**Talking points:**
- We sync inventory changes from QuickBooks to 4 external sales channels.
- Fulfillment is a child entity — we receive fulfillment events, but have to wait for product/variant entities to be resolved first before we can link them. This ordering dependency was a key design constraint.
- Records are partitioned by `realm_id` with 2 shards, which gives us horizontal scale per tenant.
- System is designed for idempotent consumers and safe recovery under partial failures — if something fails mid-sync, we don't double-process or lose events.

---

### Throughput Validation for GA

**Resume bullet:** Validated 75 TPS throughput for GA (against a 70 TPS traffic-estimate baseline) using custom datagen scripts.

**Talking points:**
- We didn't have a validated throughput number before GA — I built datagen scripts to simulate production-like load.
- The pipeline has a dependency chain: products → variants → fulfillments. I had to wait for upstream ingestion to complete, then run fulfillment ingestion, and compare published vs consumed event counts to confirm accuracy.
- Target was 70 TPS based on traffic estimation. We validated 75 TPS and signed off for GA.
- I configured Kubernetes HPA based on Kafka consumer lag — pods scale up when lag grows, scale down when it clears. This lets the system handle variable load without over-provisioning.

---

### 30s → 785ms API Timeout Fix (Catalog Variant Bulk Matching)

**Resume bullet:** Diagnosed 30-second API timeout in catalog variant bulk-matching; response time dropped from 30s to 785ms (~97%).

**Talking points:**
- The feature was a bulk match screen where users match incoming sales channel variants to existing QuickBooks catalog variants.
- Root cause: catalog variants and variants were being loaded **one-by-one** (N+1 pattern), and the catalog variant fetch was **overfetching** all associated entities even when they weren't needed.
- How I found it: enabled Hibernate query logs, surfaced top tables by query count and p90 latency, then drilled into top queries by count to find the slow ones.
- Ran `EXPLAIN ANALYZE` on the SQL to inspect query plans — found missing indexes and sequential scans where index scans should have been.
- Fixed by batching loads, eliminating unnecessary entity joins, and adding targeted indexes.
- Result: 30 seconds → 785ms. This resolved a prod timeout that was blocking users.

---

---

## Intuit — Shipping Module

**Resume bullet:** Designed account and credential management for QuickBooks shipping module; launched to GA as the first core revenue product in the Commerce Business Unit.

**Talking points:**
- I owned the design for account and credential management within the shipping feature — how QuickBooks customers connect their shipping accounts and how credentials are stored and used securely.
- We integrated with a carrier aggregator (a third-party shipping provider) so customers can fulfill invoices and print labels directly from QuickBooks.
- This was the **first revenue-generating product in the Commerce Business Unit** — a meaningful milestone for the team.
- Post-GA, hundreds of shipping labels were being created by customers.
- If asked why a carrier aggregator: it lets us support multiple carriers (USPS, UPS, FedEx, etc.) through a single API integration rather than building individual carrier integrations.

---

---

## Intuit — Credential Store

**Resume bullet:** Designed and built a credential store managing thousands of shipping API credentials; AES-256-GCM envelope encryption via AWS KMS, 30-day root key rotation, mTLS in transit.

**Talking points:**

**Why we built our own instead of using the platform:**
- Our shipping provider uses a hierarchical key model: 1 parent key for Intuit, 1 child key per realm (each company using the shipping feature). This is not OAuth.
- Our platform team's credential management only supports OAuth. We needed their support to handle this custom auth model, but their timeline to commit didn't align with our GA release date.
- Decision: build it as an isolated module within our service, with a clean interface so it can be migrated to the platform system once they support this pattern. Deliberate "build to migrate" — not a hack.

**Technical design:**
- Encryption: AES-256-GCM symmetric encryption (envelope encryption pattern).
- AWS KMS for root key management — root key never leaves the KMS keystore.
- A data encryption key is derived from the root key and used to encrypt the actual credential data.
- Root key is rotated every 30 days. Encrypted records carry a key-version header so we can decrypt with the correct key version even during/after rotation.
- Data encrypted at rest (KMS), encrypted in transit via mTLS over service mesh HTTP.
- Scale: thousands of credentials (one child key per connected QuickBooks company).

**Business impact:**
- This unblocked our GA launch on schedule.
- Enabled a revenue-sharing model with the shipping provider.
- Customers connecting through QuickBooks get bulk-discounted shipping label rates — a direct customer value add.

**If asked about security tradeoffs:**
- The isolated build was pragmatic but not a shortcut — we matched or exceeded what the platform would have provided (KMS, rotation, mTLS).
- The migration path is clean: the abstraction boundary is well-defined.

---

---

## TriNet — Authorization Service

**Resume bullet:** Built ACL-based authZ service used by 20 microservices; two-tier Redis caching eliminated per-request DB roundtrips, significantly reducing p90 latency.

**Talking points:**

**How it worked before:**
- Every authorization request hit the database twice: once to fetch employee roles, once to fetch endpoint role requirements. Both matched in code to return the authZ decision.

**What I built:**
- Two-tier Redis caching:
  - Employee roles: cached as `companyId:employeeId = [comma-separated roles]`. Evicted on expiry or on role-change events (event-driven invalidation).
  - Endpoint role requirements: cached at a longer TTL since these rarely change.
- AuthZ decision made once from DB, all subsequent calls served from cache.
- Scale: 20 microservices depended on this service.

**Dynamic policy dashboard:**
- Companion service that lets DevOps update which roles are required for each endpoint without a code redeploy.
- Primary use: enabling new endpoints during deployments and updating role requirements as access policies evolve.
- Occasional use: disabling endpoints (access control without a deploy).
- Decoupled policy changes from the deployment cycle — ops could act immediately.

**If asked about latency numbers:**
- Don't have exact p90 before/after. The improvement was meaningful enough to be the primary motivation for the caching work — the service was on the hot path for every API request across 20 services.

---

---

## TriNet — SSO Microservice

**Resume bullet:** Built core SSO microservice serving ~400k customers across ~20 cloud software provider integrations.

**TriNet context:**
- TriNet is a PEO (Professional Employer Organization). ~20 cloud software providers (Slack, GSuite-type tools etc.) are configured in TriNet's IAM for SSO.
- Target customers: small/mid-size companies using TriNet — no dedicated IT staff, no resources to manage employee password resets or configure SSO themselves.
- All TriNet clients (~400k customers at the time) were offered this service.

**What the microservice does:**
1. **SAML attribute bridge:** SAML SSO providers require specific attribute values in the XML payload to provision/authenticate users. These values are often not in the IAM. This service resolves and provides those provider-specific attributes to complete the SSO handshake.
2. **App link provider:** Returns the list of SSO-enabled applications provisioned for a specific user (the nav links they see).
3. **Handshake orchestration:** When a user clicks an SSO link, the service returns the final handshake URL to complete the SSO flow.

**Why it matters:**
- Without this, onboarding a new cloud software provider required manual configuration and had gaps where SSO couldn't complete.
- Employees get password-free access to all ~20 tools — no IT ticket needed.
- Serves every TriNet client — 400k customer scale.

**If asked about the "self-service" framing:**
- The service is what makes SSO seamless and self-completing for end users. The "self-service" benefit is that employees don't need IT to reset passwords or set up SSO — it just works when they log in.

---

---

## TriNet — Audited Messaging Framework + Admin Dashboard

**Resume bullet:** Built event-bridge processing 300–400K events/day between on-premise HR engine and microservices; admin dashboard saves 10–15 min per incident.

**Context:**
- TriNet was decomposing non-core operations out of its legacy on-premise HR engine (do not name the software) into microservices.
- These systems needed to stay in sync for critical HR operations. Data contract mismatches and system issues caused intermittent publishing/consumption failures.

**What I built:**
1. **Audited event-bridge:** All events between the two systems are written to an audit database before publishing (write-ahead). This serves as a replay buffer — if the messaging system is down or unreachable, events can be retried from the audit store.
2. **Status tracking:** Every event's status is tracked end-to-end — published, consumed, failed, retried. Enables debugging data inconsistencies between the two systems.
3. **Admin dashboard:** UI for support/admin staff to:
   - Query events and their statuses
   - Correct event data (fix bad payloads)
   - Retry/resubmit failed events
   - Full audit trail of manual changes: who changed what, when (captures human interventions)

**Impact:**
- Before: support staff ran custom SQL queries manually — required write access to the DB, error-prone, no audit trail.
- After: UI-based, audited, accessible without raw DB access. Saves ~10–15 min per issue.
- Volume: 300–400K messages per day between the two systems.

**If asked why not use a standard DLQ:**
- The need was more than just retry — we needed to *correct* the data and track human changes, which a standard DLQ doesn't support. The audit trail was also important for root-cause analysis during the decomposition.

