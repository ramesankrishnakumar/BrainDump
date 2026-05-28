# Practice Questions

Ten worked system design problems, ordered **easy → hard**. Each is a full deep-dive that follows the same template — and each opens with a **Refresher TL;DR** (the 5 key decisions + the final architecture diagram) so you can re-skim it in two minutes before an interview.

!!! tip "How to study these"
    **First pass (learning):** read a question top-to-bottom and try to design it yourself *before* reading each section. **Refresh pass (night before):** read only the **Refresher TL;DR** at the top of each page. The deep dives exist to teach you the *why*; the TL;DR is what you recall under pressure.

<!-- SECTION: the-template -->

## The template every question uses

| Step | What it covers |
|---|---|
| **0. Refresher TL;DR** | The 5 key decisions + final architecture diagram — for fast review |
| **1. Clarify & Requirements** | Functional + non-functional + explicit scope cuts |
| **2. Estimation** | Peak QPS, storage, bandwidth — only what changes the design |
| **3. API Design** | The contract |
| **4. Data Model** | Entities + storage choice (with justification) |
| **5. High-Level Design** | Boxes & arrows, happy path |
| **6. Deep Dives** | The 2-4 bottlenecks that make *this* problem hard |
| **7. Scaling & Failure Modes** | What breaks at 10x and how it fails |
| **8. Operational Excellence & Incident Response** | SLOs, alerting, graceful degradation, incident playbook — the staff operations signal |
| **9. Senior vs Staff Talking Points** | What elevates the answer |
| **10. Review Checklist** | Self-quiz |

This mirrors the [Interview Playbook flow](../interview-playbook/attacking-the-interview.md). Internalize the template and any unseen question becomes "fill in the blanks."

<!-- SECTION: the-ladder -->

## The question ladder

| # | Question | Difficulty | Core concepts it exercises |
|---|---|---|---|
| 1 | [URL Shortener (Bitly)](url-shortener.md) | ★ Easy | Key generation, [read scaling](../patterns/scaling-reads.md), [KV stores](../key-technologies/datastores.md), caching |
| 2 | [Pastebin](pastebin.md) | ★ Easy | [Blob storage](../databases/blob-storage.md), metadata/blob split, TTL/expiry, CDN |
| 3 | [Rate Limiter](rate-limiter.md) | ★★ Easy-Med | Token bucket, [Redis atomics](../key-technologies/caching.md), [contention](../patterns/contention.md), distributed counters |
| 4 | [Web Crawler](web-crawler.md) | ★★ Medium | [Queue + workers](../patterns/long-running-tasks.md), BFS frontier, dedup (bloom filter), politeness |
| 5 | [News Feed (Twitter)](news-feed.md) | ★★★ Medium | [Fan-out write vs read](../patterns/scaling-writes.md), celebrity problem, feed cache |
| 6 | [WhatsApp / Chat](whatsapp.md) | ★★★ Medium | [WebSockets](../foundations/http-and-realtime.md), message queue, delivery/read receipts, presence |
| 7 | [YouTube / Video Streaming](youtube.md) | ★★★★ Med-Hard | [Multipart upload](../patterns/large-blobs.md), [transcoding pipeline](../patterns/long-running-tasks.md), CDN, [view counts](../patterns/scaling-writes.md) |
| 8 | [Uber / Lyft](uber.md) | ★★★★ Hard | Geospatial indexing, matching, live location, surge pricing |
| 9 | [Google Drive / Dropbox](google-drive.md) | ★★★★ Hard | File sync, chunking + dedup, [metadata/blob split](../databases/blob-storage.md), conflict resolution |
| 10 | [Ticketmaster](ticketmaster.md) | ★★★★★ Hard | [Contention / reservation locks](../patterns/contention.md), inventory, read spikes, virtual waiting queue |

<!-- SECTION: cross-cutting -->

## Cross-cutting themes (appear in most questions)

- **Read vs write scaling** — almost every question hinges on which dominates. Know the [read](../patterns/scaling-reads.md) and [write](../patterns/scaling-writes.md) escalation ladders cold.
- **The metadata + blob split** — anytime files are involved (Pastebin, YouTube, Drive), the DB holds pointers and S3 holds bytes. See [Blob Storage](../databases/blob-storage.md).
- **Idempotency** — anywhere there's a queue, retries, or money, consumers must be idempotent. See [Multi-step Processes](../patterns/multi-step-processes.md).
- **Consistency choice** — say out loud where you accept eventual consistency (likes, feeds, view counts) and where you demand strong (payments, inventory, seats). See [CAP & PACELC](../distributed-systems/consistency-cap-pacelc.md).
- **Failure modes** — for every component, name how it fails and how you degrade. See [Stability Patterns](../resilience/stability-patterns.md).
