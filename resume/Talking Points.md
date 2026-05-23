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

## Intuit — AI Onboarding Agent (NTTF / QBO Trial Customers)

**Resume bullets:** Multi-agent onboarding assistant, LangGraph, context engine, 5% → 50% ramp.

**Context / Why:**
- NTTF = New To The Franchise. New QBO customers in their 30-day trial subscription.
- QBO has a "Guided Setup" — 7 modules a human expert walks through with the customer (1.5+ hour session). Goal: make this self-serve via AI to improve trial-to-paid conversion and reduce reliance on expensive expert sessions.
- Inspired by Google AI Studio. Started as a hackathon with a Senior Staff engineer; I drove it forward as primary designer/developer after they moved to other priorities. A Staff IC joined the team later — I was the tech lead throughout.

**System Architecture — 4 components:**

1. **Context Engine** (frontend, injected into QBO)
   - Agents guide customers through tasks ("click Reports in the left sidebar") — so the agent needs to know where the customer is in QBO and what they see on screen.
   - V1: Injected html2canvas to capture the screen, sent the image to a Vision LLM to generate a text description of the current page state. Added meaningful latency to every agent response.
   - Optimization journey (V1.1–V1.4): debounce (capture only after idle interval), image format/resolution experiments, masking sensitive data on screen, identified + fixed a memory leak (canvas element not being GC'd — fixed by cloning the canvas to a local variable before processing), page-settle detection (wait for navigation to complete before capturing).
   - V2: Full rewrite using DOM parser — extracts structured page data directly from the DOM tree. Faster, no Vision LLM call needed for context. Tradeoff: loses image/visual elements on screen.
   - V2.1: Agent can explicitly request a screen capture when it needs visual context, rather than capturing on every turn.

2. **Intent Orchestrator** (LangGraph)
   - Classifies each customer message: (a) onboarding question → which of the 7 modules?, (b) general QBO support question, (c) customer confused/overwhelmed → escalate to human expert.
   - Routes to the appropriate sub-agent or triggers expert handoff.

3. **Onboarding Sub-agents** (LangGraph)
   - Started with 1 module: Reporting onboarding. Templatized the architecture to support all 7 guided-setup modules.
   - Each sub-agent guides customers through setup tasks for its module using the context engine's page state.

4. **Expert Handoff Module**
   - When intent orchestrator signals confusion or customer explicitly asks for help, routes to a live human expert.
   - Critical business requirement: must not cannibalize the human expert tier (a higher-paid service offering).

**Orchestration Architecture Evolution:**
- V1: Rule-based linear flow — rigid, could not handle natural conversation deviations.
- V2: LangGraph orchestrating agent — delegates to sub-agents, session-history aware. Enabled natural conversation patterns (customers could jump between topics).
- V3: Slim, focused workflows with defined levels — reduced complexity after data showed customers were overwhelmed by the 7-module scope.

**Model:**
- OpenAI throughout the experiment.
- Intuit Film (internal platform LLM) and Gemini were evaluated as alternatives but not implemented before the experiment was shut down.

**Results & Experiment:**
- Ramped from 5% → 50% of NTTF customers during the 30-day trial window.
- **Single-module phase** (Reporting only, custom CUI): ~50% module completion rate; multi-session engagement (customers returned across multiple sessions). Even though we advertised "Reporting onboarding only," customers started asking general QBO questions — strong signal of engagement and trust.
- **7-module phase** (shared Intuit CUI): lower completion rate, customers completing modules out of order (actually a positive signal — shows natural conversation flow, not rigid script-following), customers felt overwhelmed by scope.
- **Pivot**: moved to smaller "high journey" modules — focused on fewer, high-value tasks like setting up bank connection and generating first report.
- **Key business metric passed**: did NOT cannibalize human expert tier — AI agent users continued interacting with human experts at normal rates.
- **Unproven metric**: Larger Ecosystem Revenue (LER) — could not prove AI onboarding drove higher product adoption/revenue within the trial window.
- Experiment shut down, agent decommissioned.

**How to frame the shutdown:**
- This was a data-driven decision, not an engineering failure. The system performed well technically — 50% completion, multi-session engagement, natural conversation flow.
- The hard part was the business attribution problem: proving AI onboarding drove LER within a 30-day trial window. That's a difficult measurement challenge, not a product flaw.
- The pivot to smaller modules and the eventual shutdown reflects mature product/engineering judgment: don't scale what you can't prove converts, especially when you're competing for product investment.

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

---

---

## Intuit — Live Bookkeeping Expert Workflow & Review System

**Resume bullets:** Template-driven expert workflow; configurable peer-review system; ~30 min/engagement saved across 50k active services.

**Context / Why:**
- QuickBooks Live Bookkeeping has two cleanup offerings: One-Time Cleanup (single paid engagement) and Ongoing Cleanup (monthly subscription). Both involve Intuit experts doing bookkeeping work on behalf of customers.
- Customer Service Time (CST) = total expert hours spent per engagement. It's the primary cost metric for the offering — reducing it directly reduces cost per service.

**What I built — Part 1: Expert task modeling**
- Expert work was previously unstructured. I modeled it as template-driven Projects and Tasks in the workflow engine.
- Templates are configured per service; the system instantiates them automatically at the right workflow stage (e.g., document-collection project created after the onboarding call completes).
- This gave the business visibility into where expert time was going — the first step toward reducing CST.
- I was tech lead on this, guiding 1–2 engineers through the implementation.

**What I built — Part 2: Peer review system**
- After a task is complete, an expert can request peer review. Key design decisions:
  1. **Reviewer assignment:** FIFO queue — the engagement/work request is placed into queues where leads are waiting. Ensures fair load distribution without manual assignment.
  2. **Review completion:** Reviewer fills out a structured YES/NO questionnaire and leaves notes. Review signaling cascades to update task status in the correct order.
  3. **Notifications:** Both reviewer and reviewee are kept informed throughout — reduces Slack back-and-forth.
  4. **UiPath RPA integration:** For the final review stage, UiPath automation can auto-answer certain questionnaire items, reducing lead burden.
- Before this system: review coordination was entirely via Slack/email — no audit trail, no visibility into review status, constant context switching.

**Extensibility design (senior/staff angle):**
- The review system is config-driven: extending to a new service requires only (a) declaring review types for that service, (b) mapping a questionnaire template to each review type, (c) configuring notification templates.
- No code changes needed to onboard a new service. This was a deliberate architectural decision to avoid bespoke review logic per offering.
- The complexity was service discovery and mapping — understanding which services existed and how to dynamically map them to review configurations.

**Impact:**
- Each review saves ~10 min (prevents context switching, consolidates rework context, RPA handles final-stage auto-review).
- Average 3 reviews per service × 50k active services = meaningful CST reduction at scale.
- Framing for interview: ~30 min of expert time saved per engagement across 50k active services.

**If asked about the before/after CST number:**
- Don't have a precise before/after delta. Frame it as: CST was the tracked metric the business cared about, the review system + task modeling were two levers to move it, and the 30-min/engagement figure is the review system's direct contribution.

