Krishnakumar Ramesan
Hayward, CA | (256) 652-8738 | ramesankrishnakumar@gmail.com | [linkedin.com/in/rkrrish](https://www.linkedin.com/in/rkrrish/) | [github.com/ramesankrishnakumar](https://github.com/ramesankrishnakumar)
May 29, 2026

Samsara Hiring Team
Samsara
Remote (US)

---

Dear Hiring Team,

I'm applying for the Senior Software Engineer role on the Route Execution team. Tracking and executing hundreds of thousands of routes in real time, with a dispatch state machine that has to stay correct under failure and a flood of TMS/ERP events, is squarely the kind of distributed system I've spent my 13+ years building.

The closest match to your route-execution work is a fault-tolerant fulfillment subsystem I own at Intuit. It ingests events from 4 channels at hundreds of thousands per day, and because each event must be linked to upstream entities before downstream consumers act on it, ordering and exactly-once handling are core design constraints, the same correctness bar a dispatch state machine demands. I built it for idempotent consumers, ordered delivery, and safe recovery under partial failures, then scaled it to sustain 75 TPS through partitioning, sharding, and autoscaling. When a related bulk-matching query timed out at 30 seconds, I profiled the access patterns and brought it down to 785 ms, a 97% reduction.

Earlier, at TriNet, I built an audited event-bridge connecting an on-premise HR engine to new microservices, processing 300 to 400K events per day with write-ahead logging as a replay buffer so nothing was lost when a downstream call failed. I paired it with an admin dashboard that let support query, correct, and resubmit failed events with a full audit trail, which is the single-pane-of-glass operator tooling dispatchers and fleet managers need to run their day. I design real-time pipelines for the operators who live in them, not just for the happy path.

Your stack is Go, GraphQL, and TypeScript/React. My recent backend work is in Kotlin, Java, and Python, I've built GraphQL APIs with Netflix DGS, and I've shipped operator-facing web tooling end to end, so I'll ramp on Go quickly and contribute across the stack. I care most about shipping features customers use immediately and iterating from their feedback.

I'd welcome the chance to talk about how I can help build the next generation of route execution at Samsara.

Sincerely,
Krishnakumar Ramesan
