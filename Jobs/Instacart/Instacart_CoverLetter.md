Krishnakumar Ramesan
Hayward, CA | (256) 652-8738 | ramesankrishnakumar@gmail.com | [linkedin.com/in/rkrrish](https://www.linkedin.com/in/rkrrish/) | [github.com/ramesankrishnakumar](https://github.com/ramesankrishnakumar)
May 29, 2026

Instacart Hiring Team
Instacart
United States (Remote)

---

Dear Hiring Team,

I'm applying for the Senior Software Engineer, Storage role on the Key-Value team. The work you describe, namely stewarding in-memory datastore infrastructure across hundreds of clusters with 24/7 uptime requirements, is squarely where I've spent my 13+ years: building distributed backend systems where correctness under partial failure and cost-efficient scale are the whole game.

At Intuit I own a fault-tolerant fulfillment subsystem that ingests events from 4 sales channels at hundreds of thousands per day, designed for idempotent consumers, ordered delivery, and safe recovery under partial failures. I scaled it to sustain 75 TPS through database partitioning, sharding, and load-based autoscaling. When a production query timed out at 30 seconds during bulk catalog matching, I profiled the access patterns and removed redundant round-trips, unnecessary data loads, and missing indexes, dropping response time to 785 ms, a 97% reduction. I've also shipped production multi-agent AI systems on LangGraph, which maps directly to the AI agent-based monitoring and operations the team is investing in for the future.

At TriNet I worked closest to the in-memory storage problems this role centers on. I built an ACL-based authorization service consumed by 20 microservices that eliminated per-request database round-trips with a two-tier Redis cache, namely per-user role entries invalidated on role-change events and endpoint policies held at a longer TTL, measurably reducing p90 latency under production load. I paired it with a dynamic policy dashboard so DevOps could change role requirements across all 20 services without a redeploy, which is control-plane work in spirit. I also built an audited event-bridge processing 300 to 400K events per day with write-ahead logging as a replay buffer for failure recovery.

My recent backend work is in Python alongside Kotlin and Java, so I'll be productive in your codebases quickly and am genuinely excited to go deep on Go, Rust, and Valkey internals. Beyond shipping, I care about the things you screen for: reviewing PRs, raising the bar on standards, and growing junior engineers so the whole team moves faster.

I'd welcome the chance to talk about how I can help build Instacart's storage platform.

Sincerely,
Krishnakumar Ramesan
