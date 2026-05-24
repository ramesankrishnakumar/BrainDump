At Intuit, I build backend systems where data correctness, throughput, and security are foundational requirements, not afterthoughts. That maps directly to what Account Master describes.

The clearest match is a fault-tolerant event-driven subsystem I own that ingests fulfillment events from four external sales channels, processing hundreds of thousands of events daily. The core design challenge was ordering: fulfillment events had to wait for upstream product and variant entities to be resolved before they could be linked and published for downstream inventory consumers. I built the system for idempotent consumption and safe recovery under partial failures, partitioned data by tenant for horizontal scale, and validated 75 TPS throughput before GA using custom load scripts. I configured Kubernetes HPA against Kafka consumer lag so the system scales under real load without over-provisioning.

On the security side, I designed and built a credential store for thousands of shipping API credentials when our platform team could not meet our launch timeline. I implemented envelope encryption with AES-256-GCM, root key management through AWS KMS with 30-day rotation, and mTLS in transit, built as a clean migration-ready module to hand off once the platform added support for our provider's key model.

When a bulk-match API was timing out at 30 seconds in production, I found the root cause through Hibernate query logs and EXPLAIN ANALYZE: N+1 loads and missing indexes. Batching the loads, removing unnecessary entity joins, and adding targeted indexes brought it to 785ms, a 97% reduction that resolved a live customer-blocking issue.

More recently, I drove a production multi-agent AI onboarding system from hackathon to 50% of first-time trial customers on LangGraph, including a DOM-based context engine that replaced a screenshot-plus-vision pipeline and cut per-turn latency from 9 seconds to 30ms.

Thirteen years of this, across event-driven systems, identity, security, and AI. Tesla's Account Master challenge is the kind of foundational, high-stakes work I'm drawn to.
