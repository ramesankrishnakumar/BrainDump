Krishnakumar Ramesan
Hayward, CA | (256) 652-8738 | ramesankrishnakumar@gmail.com | [linkedin.com/in/rkrrish](https://www.linkedin.com/in/rkrrish/) | [github.com/ramesankrishnakumar](https://github.com/ramesankrishnakumar)
May 29, 2026

Grow Therapy Hiring Team
Grow Therapy
Remote

---

Dear Hiring Team,

I'm applying for the Senior/Staff Software Engineer, Marketplace role on the Match Success team. A three-sided marketplace that connects clients to the right provider and keeps that relationship healthy is fundamentally a matching, routing, and trust problem at scale, and that's the kind of system I've spent my 13+ years building.

The closest match to your applied-AI and matching work is a multi-agent AI assistant I designed and shipped at Intuit, which I drove from hackathon prototype to production as tech lead. At its core is a config-driven intent orchestrator that classifies each user message and routes it, either to a handoff, a general fallback, or a second orchestrator that delegates across 7 domain agents or answers from prior context. That is a live routing and recommendation problem, and the config-driven design lets us evolve the matching logic without redeploys. I also built the screen-context engine grounding those agents in what the user sees, replacing a 9-second-per-turn vision pipeline with a roughly 30 ms DOM parse, a 300x gain I found by instrumenting and measuring the bottleneck rather than guessing.

On the marketplace-transactions side, I own a fault-tolerant fulfillment subsystem that ingests events from 4 sales channels at hundreds of thousands per day, built for idempotent consumers, ordered delivery, and safe recovery under partial failures, scaled to 75 TPS through partitioning, sharding, and autoscaling. I also built a configurable assignment system for expert work using FIFO-based reviewer matching, where new service lines onboard through config alone, saving roughly 25,000 hours of expert time. When a bulk catalog-matching query timed out at 30 seconds, I profiled the access patterns and brought it down to 785 ms, a 97% reduction.

My recent backend work is in Kotlin, Java, and Python, with full-stack reach into the DOM and front-end layer through the context-engine work, so I'll map quickly to your stack. Beyond shipping, I lead by raising the bar in code review, writing the playbooks and guardrails that keep a team consistent, and growing the engineers around me.

I'd welcome the chance to talk about how I can help build the marketplace that makes care accessible at national scale.

Sincerely,
Krishnakumar Ramesan
