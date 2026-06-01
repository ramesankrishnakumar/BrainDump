Krishnakumar Ramesan
Hayward, CA | (256) 652-8738 | ramesankrishnakumar@gmail.com | [LinkedIn](https://www.linkedin.com/in/rkrrish/) | [GitHub](https://github.com/ramesankrishnakumar)
May 30, 2026

Box Hiring Team
Box, Inc.
Redwood City, CA

---

Dear Box Hiring Team,

I'm applying for the Senior Software Engineer role on the Core Agents team. Box is betting that the combination of AI and enterprise content will reshape how companies work, and the leverage point is a platform that lets teams ship agents safely on top of LangGraph. That is the exact work I have been doing. At Intuit I designed and shipped a production multi-agent AI onboarding assistant for QuickBooks on LangGraph and OpenAI GPT 4.1, taking it from hackathon prototype to production as tech lead and ramping it to 50% of first-time trial customers. I built a config-driven intent orchestrator that classifies each user message and routes it across 7 domain agents, which is the kind of framework, tooling, and orchestration pattern your Agents Platform exists to standardize.

Your platform also has to enforce tenant isolation, data governance, and least-privilege access, and that has been a throughline of my work. I secured thousands of carrier API credentials with AES-256-GCM envelope encryption via AWS KMS, versioned root key rotation, and mTLS in transit, built as an isolated module to unblock a GA launch. Earlier I built a core SSO microservice serving roughly 400K customers across about 20 SaaS integrations and an ACL-based authorization service consumed by 20 microservices, enforcing per-role, per-endpoint access with a dynamic policy dashboard so access rules change without code redeploys. For the AI work I built a screen context engine that redacts SSNs, addresses, and phone numbers on-page before any model sees them, so governance is handled at the source.

The platform promise of low-latency, high-throughput execution with real observability is where I spend most of my time. I owned a fault-tolerant fulfillment subsystem ingesting events from 4 sales channels at hundreds of thousands per day, scaled to sustain 75 TPS through partitioning, sharding, and load-based autoscaling, with idempotent consumers and ordered delivery for safe recovery under partial failure. When that same agent context pipeline ran a screenshot-plus-vision-LLM step at roughly 9 seconds per turn, I replaced it with a 30 ms DOM parse. I work primarily in Kotlin, Java, and Python on distributed systems built for reliability at scale.

I'd welcome the chance to help build the platform that lets Box and its customers ship agents quickly and safely. Thank you for your consideration.

Sincerely,
Krishnakumar Ramesan
