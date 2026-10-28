# System Design — Quick Notes (Sections 1–2)

Source: Udemy *System Design & Architecture Masterclass + Case Studies* — Section 1 (Introduction, 7 lectures) and Section 2 (Building Blocks of Scalability, 8 lectures).
Read top to bottom. Each lecture ends with a "Remember" line. Items marked **[fix]** are spots where the lecture was wrong or sloppy and the correct value is given.

**Contents**
- Section 1: [1 Why it matters](#1-why-system-design-matters) · [2 Think like an architect](#2-think-like-an-architect) · [3 Functional reqs](#3-functional-requirements) · [4 Non-functional reqs](#4-non-functional-requirements) · [5 Estimation: storage/bandwidth](#5-back-of-envelope-part-1--storage--bandwidth) · [6 Estimation: QPS/servers](#6-back-of-envelope-part-2--qps--servers) · [7 4-step framework](#7-the-4-step-design-framework)
- Section 2: [9 Vertical scaling](#9-vertical-scaling) · [10 Horizontal scaling](#10-horizontal-scaling) · [11 Load balancing basics](#11-load-balancing-basics) · [12 LB algorithms 1](#12-load-balancing-algorithms--part-1-round-robin) · [13 LB algorithms 2](#13-load-balancing-algorithms--part-2-least-connections-ip-hash-consistent-hashing) · [14 Stateless design](#14-stateless-service-design) · [15 Service discovery](#15-service-discovery) · [16 Circuit breakers](#16-circuit-breakers--resilience)
- [Cheat sheet](#cheat-sheet-numbers-to-memorise)

---

# SECTION 1 — Introduction

## 1. Why System Design Matters
- Coding = *how* to implement. System design = *what* to build and *why*.
- Same code that works for 100 users breaks at 1M. Example: all posts in one table → 1M posts/day → 30M/month → every query slow. That is a **design** problem, not a code problem.
- AI can generate code, but cannot decide architecture: SQL vs NoSQL, when to cache, how to keep data consistent. Those need trade-off, business and cost judgement.
- Interviews: juniors get DSA; seniors get "design WhatsApp / YouTube". Asked for architecture, not code.
- Career: junior implements a given architecture; senior/lead makes the architecture decisions (e.g. notifications: real-time or delayed? WebSockets or polling? offline users?). This is where the big pay gap is (claim: 2–3x).
- Course path: fundamentals (scalable, reliable) → components (DB, cache, queue) → case studies (Instagram, Uber) → always think in **trade-offs**; there is rarely a perfect answer.

**Remember:** system design is the skill AI can't replace; it's about trade-offs.

## 2. Think Like an Architect
Developer asks "how do I build this?" — architect asks "what happens at a million users, what does it cost over 5 years, can 50 devs maintain it?"

Six mental shifts:
1. **Trade-offs, not perfection.** Payments → pick consistency (small error = big problem). Social feed → pick availability (stale feed beats no feed).
2. **Think about failure.** For every component ask: what if it fails, how do we recover? (DB crash, slow network, data centre down, attack). E-commerce payment down at checkout → options: show error (lose sale), save order and charge later, or fail over to backup provider.
3. **Components + interfaces, not implementation.** "Upload photo" = upload service + storage service + image-processing service. Think how they talk, what if one is slow, scale each independently. Modular → replace one without touching others.
4. **Cost.** Microservices = more servers, monitoring, complexity. Example: 1 TB logs/month in DB ≈ $500; cold storage after a week ≈ $50 → saves $5,400/yr. Tier data (hot/cold/archive).
5. **Whole lifecycle.** Evolution, schema migrations, tech debt. Example: MongoDB fine now, but complex reporting in 2 years may force a costly migration to relational.
6. **Patterns / anti-patterns.**
   - Single point of failure → fix with redundancy (backup DBs, load balancers, multiple servers).
   - Tight coupling (change one service → change five) → fix with loose coupling (queues, API contracts, service boundaries).

**Remember:** trade-offs · failure scenarios · components · cost · lifecycle · patterns.

## 3. Functional Requirements
- **What the system should do** — features users directly use.
- "Build a chat app" is an idea, not a requirement. Real ones: send text, see delivery status, group chats, searchable messages.
- **Dig into each one.** "Send messages": text only? images/video/files? max size? expiry? editable? Supporting video forces storage + transcoding + streaming.
- **Technique: start from user stories / user types.** E-commerce → buyers (search, cart, pay, track), sellers (list, inventory, reports, returns), admins (users, transactions, analytics, disputes).
- **Scope & MVP.** Separate must-have vs nice-to-have. YouTube v1 = upload + playback; comments, live, monetisation later.
- **Make them measurable/testable.** Not "upload quickly" → "100 MB video uploads in ≤2 min on standard broadband". Not "good search" → "relevant results in <200 ms for 95% of queries".
- **Hidden requirements people forget:** delete (what happens to comments?), privacy (who sees what), blocking (what does the blocked user see?). Missing these causes redesigns.
- Ride-share edge cases: driver cancels after accepting, payment fails after ride, GPS lost mid-ride, surge pricing, scheduled rides.
- **Constraints inside requirements:** "10M active users" → distributed, load balancing, sharding. "End-to-end encrypted" → can't store plaintext, no server-side search.
- Requirements evolve (text → emoji → images → video → stories). Design so you aren't locked in, but don't over-engineer.

**Remember:** specific, measurable, cover all user types + edge/failure cases, prioritised. Missing one can mean a full redesign.

## 4. Non-Functional Requirements
- **How well** the system does it (quality attributes). Instagram: functional = upload photos; non-functional = upload <3 s (performance), 500M uploads/day (scalability), never lose a photo (reliability), 99.9% uptime (availability).
- **Performance** — latency/response time. Amazon: +100 ms latency ≈ −1% sales. Google: +500 ms ≈ −20% traffic. Targets: page <2 s, API <200 ms, DB query <100 ms.
- **Scalability** — handle growth. Vertical (bigger machine) vs horizontal (more machines). Netflix: ~200M users, traffic spikes 10x on releases, auto-scales.
- **Availability** — "nines":
  | Nines | Uptime | Downtime / year |
  |---|---|---|
  | 2 | 99% | 3.65 days |
  | 3 | 99.9% | 8.76 hours |
  | 4 | 99.99% | 52 minutes |
  | 5 | 99.999% | ~5 minutes **[fix: lecture said "99.99%" for 5 nines]** |

  Each extra nine costs exponentially more (3→4 nines can double infra cost). Choose what the business needs.
- **Reliability** — correct behaviour even when things fail. Amazon cart replicated across servers/data centres.
- **Security** — authentication (who), authorisation (what allowed), encryption (in transit + at rest), audit logging (who did what, when). Breaches cost trust and billions in fines.
- **Maintainability** — documented, add features without breaking others, quick bug fixes, teams work independently.
- **Usability** — intuitive, helpful errors, works on all devices (Google search = one box, one button).
- **Operational** — monitoring, logging, backup/recovery, low-downtime deployment.
- **They conflict:** more availability = more redundancy = more cost; more performance may trim security checks; perfect consistency may cost availability (**CAP theorem**).
  - Bank → security + consistency over speed. Social feed → performance + availability over consistency.
- Always state numbers: "200 ms for 99% of requests", "100,000 concurrent users with <1% errors".

**Remember:** specific + measurable; know which attribute wins for this product.

## 5. Back-of-Envelope Part 1 — Storage & Bandwidth
**Why:** quickly estimate storage, bandwidth, servers — no calculator, in interviews/planning. Aim for order of magnitude ("approximately right beats precisely wrong"). Round aggressively.

**Units (powers of 10):** 1 KB = 10³ B · MB = 10⁶ · GB = 10⁹ · TB = 10¹² · PB = 10¹⁵. Millions = 10⁶, billions = 10⁹, trillions = 10¹².

**Typical sizes:** 1 char = 1 B · tweet ≈ 300 B (with metadata) · phone photo ≈ 4 MB (3–5) · HD video ≈ 50 MB/min · 4K video ≈ 300 MB/min.

**Twitter storage:** 500M tweets/day × 300 B = 150 GB/day text. If 10% have a 4 MB image → 50M × 4 MB = 200 TB/day. **Media dominates storage.**

**Bandwidth:** measured in **bits**/s; 1 byte = 8 bits (1 MB/s = 8 Mbps). Home ≈ 100 Mbps, data-centre link ≈ 1 Gbps, backbone 10–100 Gbps. 1 day ≈ 86,400 s → use 100,000.
- YouTube example (lecture): 1B hours watched/day = 60B minutes. **[fix: lecture mislabels HD as "50 MBPS"; it is ~50 MB per minute of video]** → ~3,000 PB/day → ÷100,000 s ≈ 30 TB/s ≈ 240 Tbps just for delivery.

**Rules of thumb**
- Peak traffic = **2–3x average**.
- **80/20 rule:** 20% of content gets 80% of traffic → cache popular content.
- **Replication:** keep ~3 copies → multiply storage by 3.
- **CDN/cache hit ratio:** at 80% hit, only 20% of requests reach origin.
- Instagram: 500M DAU × 2 photos × 4 MB = 4 PB/day ≈ 1.5 EB/year; 5 years ≈ 7.5 EB; ×3 replication ≈ 22.5 EB (excluding video, stories, backups).

**Remember:** media dominates; bits vs bytes; ×3 for replication; peak = 2–3x avg.

## 6. Back-of-Envelope Part 2 — QPS & Servers
- **QPS** = total daily requests ÷ 86,400 (≈100,000). Peak QPS = 2–3x average (events can be 5–10x).
- **Twitter:** 300M DAU × 100 tweets read = 30B reads/day → **300K read QPS** avg → ~900K at peak. Writes: 500M/day → **5K write QPS** (~15K peak). Read:write ≈ **60:1** → optimise for reads.
- **Per-server capacity (rough):**
  - Serving cached data: 50K–100K QPS
  - API calls with some logic: 5K–10K QPS
  - Heavy (DB queries, video processing): 100–1K QPS
  - 1 CPU core ≈ 1K–10K simple ops/s; 32 cores ≈ 30K–300K ops/s; NIC 10–25 Gbps; SSD ≈ 100K IOPS vs HDD 100–200 IOPS.
- **Netflix streaming:** 200M subs, 25% concurrent = 50M streams; 10K streams/server → 5,000 servers; +20% redundancy (N+1) = 6,000; +10% failure buffer = 6,600. Plus auth (~1M QPS ÷ 10K = 100 servers), recommendations (~500), etc.
- **Utilisation target 60–70% CPU**, never 100%. 100 servers at full load → 140–150 at 70%.
- **Database:** Postgres ≈ 100–200 connections; at 100 ms/request each connection does 10 req/s → ~2,000 QPS/DB. 100K QPS would need ~50 DBs → impractical → use **connection pooling, caching, read replicas**.
- **Cache power:** 90% hit rate turns 100K QPS into 10K at DB → 5 DBs instead of 50.
- **Geography:** split by traffic share (e.g. 30% US, 25% EU, 35% Asia, 10% other), add ~20% redundancy per region.
- **Cost:** cloud server ≈ $100–500/mo; 1,000 servers ≈ $100K–500K/mo; ~$1M/mo with bandwidth/storage. 20% efficiency gain ≈ $2.4M/yr.
- **Interview tip:** state ideal first, then reality: "10K QPS/server → 100 servers; at 70% utilisation + 20% redundancy → 150."

**Remember:** QPS = daily ÷ 100K; peak ×2–3; 70% utilisation; cache slashes DB load.

## 7. The 4-Step Design Framework
Time split for a 45–60 min interview: **10 / 15 / 20 / 15**. Better to cover all four briefly than get stuck on one. Engage the interviewer throughout ("should I go deeper here?").

**Step 1 — Requirements (5–10 min).** Never design before this. Clarify functional (Twitter: post, follow, timeline, like/retweet), non-functional (e.g. 100M DAU, 500K read / 5K write QPS, 200 ms timeline, 99.9%), and constraints (global/regional? media or text only? retention? budget?). Write them down.

**Step 2 — High-level design (10–15 min).** Bird's-eye boxes and data flow, no schemas yet: clients → CDN (static) → load balancer → app servers → cache → DB; media → object storage (S3); timeline as a separate service with its own cache; message queues for async. Walk the core flows (post tweet, build timeline, follow).

**Step 3 — Detailed design (15–20 min).** Deep-dive the critical parts.
- Timeline: **pull** (build on request: cheap writes, slow reads) vs **push** (precompute on post: costly writes, fast reads). Twitter is **hybrid** — push for normal users, pull for celebrities.
- Schema: users, tweets, follows (follower_id, followee_id **[fix: lecture said follower/follower]**), likes. Indexes: user_id (user's tweets), timestamp (ordering), composite (follower, timestamp) for timeline queries.
- Justify tech: Postgres (ACID, complex queries), Cassandra (high write throughput, time-series), Redis (ms latency, data structures).

**Step 4 — Scale & optimise (10–15 min).**
- Find bottlenecks and fix: DB writes → shard by user ID; heavy timelines → cache + precompute; media → CDN + compression; hot partitions (celebrities) → consistent hashing + replicas.
- Monitoring: QPS, latency, error rate, cache hit ratio; alert on spikes.
- Failures: data centre down → regional failover; cache down → degrade to DB; DDoS → rate limiting + traffic filtering.
- Cost: spot instances for batch, compress old data, lifecycle/archival policies.

**Remember:** Requirements → High-level → Detailed → Scale. Don't skip step 1.

---

# SECTION 2 — Building Blocks of Scalability

## 9. Vertical Scaling
- **Scale up** = make the existing server bigger (more RAM/CPU). No code change.
- Example: 8 GB/2 cores struggles at 2K users; move to 64 GB/16 cores → handles 10K.
- **AWS cost curve (not linear):** t2.micro 1 GB/1 vCPU ≈ $8/mo · t2.large 8 GB/2 CPU ≈ $67 · m5.24xlarge 384 GB/96 CPU ≈ $4,600 · x1e.32xlarge ~3.9 TB/128 CPU ≈ $26,688. Top-end hardware is rare and disproportionately priced.
- **Hard limits:** Moore's Law slowing (transistors near atomic scale); biggest box ~448 cores. Facebook ~4M likes/s × 1 ms = 4,000 cores for likes alone — impossible on one machine.
- **When it's right:** small/medium apps; **databases early on** (distributed DBs are complex — one big DB e.g. 512 GB/64 cores handles millions of queries/day, shard only at true scale); legacy/stateful software that doesn't distribute. Stack Overflow: ~9 web servers for ~50M devs (big machines, aggressive caching, DB in RAM).
- **Pros:** no code changes, no distributed complexity, no network hops between components, simpler backup/monitoring/debugging, instant boost.
- **Cons:** hardware ceiling, **single point of failure**, expensive at high end, downtime for upgrades, vendor lock-in, no geographic distribution.
- Cautionary tale: **Friendster** kept buying bigger servers, hit the wall in 2003 (40 s page loads), users fled to MySpace/Facebook.
- **Switch to horizontal when:** upgrade cost > adding servers, need geo-distribution, downtime unacceptable, near hardware limits, need fault tolerance. Typical path: vertical first → hit limits at ~100K–1M users → go horizontal, keeping DBs vertical longer.

**Remember:** simple and fast to start, but capped, costly, and a single point of failure.

## 10. Horizontal Scaling
- **Scale out** = add more same-size (commodity) servers.
- Amazon story: ~2000 ran on huge $100K+ Sun servers; chose ~1,000 cheap ($2K) Linux boxes by 2001 → similar cost, infinite scalability → became AWS.
- **Key enabler = stateless design** (details in lecture 14). Stateful: session in one server's RAM → request must return to that server; crash loses sessions. Stateless: server issues a **JWT**, remembers nothing; any server can verify it. Cart → DB, session → Redis, files → S3.
- **Linear scaling:** Netflix API server ≈ 20K req/s → 10 servers = 200K, 100 = 2M. Pandemic traffic +60% in two weeks, just added instances.
- **Cost:** one x1e.32xlarge ($26,688, ~50K req/s) vs ~267 t3.large at ~$100 each (~2K req/s each → ~534K req/s) → **>10x capacity for same money**, and you can scale down when quiet.
- **Fault tolerance:** Google: ~20 failures/day per 10,000 servers. With 1,000 servers, losing one = 0.1% capacity; LB routes around it. Netflix **Chaos Monkey** kills servers in production on purpose to prove resilience.
- **New problems you must solve:**
  - Load balancers (even distribution, health detection, adding servers)
  - Service discovery (which of 50 payment instances to call?)
  - Centralised logging (which of 100 servers handled this request?)
  - Data consistency / race conditions across servers
  - Code changes: file uploads → shared storage (S3); scheduled jobs → distributed scheduling so a job runs once not 20 times; caching → shared (Redis/Memcached), not per-server memory
- **Uber:** 4,000+ microservices, hundreds of thousands of containers, every service stateless and independently scalable; New Year's Eve ×10 auto-scaled.
- **Auto-scaling:** e.g. AWS adds instances when avg CPU >70%, removes when <30%. Spotify scales with commute/night patterns, saves millions.
- **Geo-distribution:** servers near users — latency 200 ms → ~20 ms (Facebook: data centres in ~15 locations).
- **Choose horizontal from day one if:** rapid growth, high availability, geo-distribution, spiky traffic, cloud-native. **Start vertical if:** prototype, complex stateful logic, legacy code, small stable user base, max single-thread performance.
- Best practice: **app servers horizontal (stateless), databases vertical as long as possible.**

**Remember:** stateless + LB + shared state stores = linear scaling and built-in fault tolerance, at the price of distributed-system complexity.

## 11. Load Balancing Basics
- **Load balancer (LB):** single entry point that spreads requests across servers so none is overloaded while others idle.
- **Why not DNS round robin?** DNS can't see server health/load, changes propagate slowly (hours), and ISP caching can pin millions of users to one server (real incident: one server got 5x traffic).
- **Flow:** DNS returns the LB's IP (not a web server) → LB picks backend (e.g. server 7) → backend responds via LB → client never knows how many servers exist.
- **What it gives you**
  1. **Even distribution.**
  2. **Health checks + automatic failover:** every ~2–5 s hit e.g. `GET /health`, timeout ~200–500 ms, mark unhealthy after 2–3 failures, stop sending traffic, resume when healthy again (self-healing). Airbnb: DB slowdown made app servers hang → LB pulled them in ~10 s → site degraded, not down.
  3. **Zero-downtime deployment (blue-green):** blue = old version, green = new; test green, switch traffic, instant rollback by switching back.
- **Layers**
  - **Layer 4 (transport):** sees IP + port only; very fast (millions of connections/s); no content-based routing.
  - **Layer 7 (application):** understands HTTP; routes on URL path, headers, cookies (`/api/*` → API servers, `/images/*` → image servers, `beta=true` cookie → beta servers). YouTube uses it to route video, API, and upload traffic to specialised servers.
- **Sticky sessions (session affinity):** LB sets a cookie (e.g. `server=3`) and keeps sending that user there. Needed for WebSockets/local caches, but reduces flexibility and loses sessions if that server dies → prefer stateless.
- **Extra features:** SSL termination (backends speak plain HTTP, lower CPU), response compression, rate limiting (e.g. 100 req/s per IP, DoS protection), WAF (blocks SQL injection, XSS).
- **Scale numbers:** Netflix uses many AWS ELBs; Uber uses HAProxy (~50K req/s each, thousands of instances); Google built **Maglev** (~1M req/s per machine, uses consistent hashing).
- **LB is itself a single point of failure** → run several: **active-passive** (standby takes over in seconds) or **active-active** (DNS returns multiple LB IPs). Facebook: LBs in every data centre, layered ("load balancing all the way down").
- **Choosing:** Nginx (simple, ~10K+ req/s, ~35% of websites) · HAProxy (gold standard, 100K+ req/s) · cloud managed (AWS ALB, Google Cloud LB, Azure LB) · hardware (F5, Citrix — expensive, very high throughput).

**Remember:** LB = traffic controller + health checker + enabler of zero-downtime deploys; make the LB itself redundant.

## 12. Load Balancing Algorithms — Part 1 (Round Robin)
- The algorithm decides which server gets the next request (millions of times/s).
- **Round robin:** 1,2,3,1,2,3… Equal share per server. 9,000 req/s over 3 servers = 3,000 each. Great when servers are identical.
- **Problem:** unequal hardware. Example: two 8-core + one 32-core server → old ones at 95% CPU (2 s responses), new one at 25%.
- **Weighted round robin:** assign weight by capacity. Weights 1,1,4 (total 6) → 1,500 / 1,500 / 6,000 req/s → all ~70% CPU, ~200 ms. Implemented via a "virtual list" (A weight 2, B weight 3 → A,B,A,B,B) or counters.
- **Use cases**
  - **Canary / gradual rollout (Netflix):** new version weight 1 vs old 19 (5%) → 5 vs 15 (25%) → 10/10 (50%) → 19/1 (95%) → remove old. Instant rollback by changing weights; no redeploy.
  - **Geo-balancing (Spotify):** adjust regional weights through the day (US-morning 7/2/1 for US/EU/Asia; reverses at night).
  - **Cost mix:** on-demand weight 10 vs spot (≈70% cheaper but can vanish with 2-min notice) weight 3.
- **Limitations**
  1. Ignores actual load (3 heavy PDF jobs of 5 s each pile onto one server).
  2. Treats all requests equal (Twitter timeline vs profile view → equal count, unequal work).
  3. Doesn't adapt in real time (GC pauses, memory leaks).
- Still the default almost everywhere (Nginx, Kubernetes services, AWS ALB target groups): simple, predictable, no state, easy to debug.
- **Use RR** when servers identical + requests similar; **weighted RR** for different capacities, canary rollouts, regional balancing, mixed instance types.

**Remember:** RR = fair by count; weighted RR = fair by capacity; neither knows real load.

## 13. Load Balancing Algorithms — Part 2 (Least Connections, IP Hash, Consistent Hashing)
**Least connections** — send to the server with the fewest *active* connections.
- Example: server 1 finished fast product listings (0 conns), server 2 has 5 × 3 s checkouts, server 3 has 5 × 10 s image uploads → next request goes to server 1. Reflects real work, not request count.
- Netflix: streams vary hugely in length/bandwidth; least-connections stops any server accumulating long-lived connections.
- **Weighted least connections:** compare as % of capacity. Server A 400/1,000 = 40% vs B 900/2,000 = 45% → A gets the next one.
- Best for **long-lived connections (WebSockets)**, e.g. Slack. Plain RR over a week would pile up connections on early servers; least-connections keeps them balanced.

**IP hash** — hash(client IP) → same server every time (automatic session affinity, no cookies).
- Fits legacy in-memory sessions/carts, and GitHub git operations (long clones must stay on one backend); also helps per-IP rate limiting with local state.
- **Problems**
  1. **Uneven with NAT:** 5,000 employees behind one corporate IP all hit one server.
  2. **Mobile users change IP** (Wi-Fi → 5G) → land on a different server → session lost.
  3. **Scaling reshuffles everyone:** `hash % 5` → `hash % 6` moves almost all users → lost sessions, cache-miss storm.

**Consistent hashing** — fixes the reshuffle problem.
- Hash ring (e.g. positions 0–1M). Hash servers onto the ring; hash a client onto the ring and walk **clockwise** to the first server.
- Example: A@100K, B@400K, C@700K; client@250K → B. Add D@550K → only clients between 400K–550K move (to D); everyone else unchanged (~25% moved, not ~100%).
- Used by Cassandra, DynamoDB, Discord (chat rooms → servers; adding machines moves only a fraction), Google Maglev.

**Remember:** least-conn = real load; IP hash = stickiness but fragile; consistent hashing = minimal movement when servers change.

## 14. Stateless Service Design
- **State** = anything a server remembers between requests (in-memory sessions, cart in local variables, files on local disk, per-user local cache). Stateful: "I remember you." Stateless: "Tell me everything each time."
- **Stateful flow:** login on server 5 → session in 5's RAM → cookie with session ID → next request *must* hit server 5, otherwise "invalid session, log in again."
- **Stateless flow:** login → server returns a signed **JWT** (user ID, permissions, cryptographically signed, tamper-proof) → forgets you → next request (to server 8) carries the token → server 8 verifies the signature, reads the user ID, fetches data from the DB, serves it. Servers never talk to each other.
- **Benefits**
  1. **Perfect horizontal scaling** — add servers instantly, no warm-up (Netflix: 10K → 15K instances for Sunday episode drops).
  2. **Failure resilience** — nothing to lose when a server dies (Twitter loses hundreds of servers/day unnoticed).
  3. **Simple deployments** — drain, replace, no session migration (Facebook deploys thousands of times/day).
- **Where state goes**
  | State | Goes to |
  |---|---|
  | Sessions | Redis / Memcached (key like `session_12345`) or JWT (no server storage) |
  | User data | Database |
  | Uploaded files | Object storage (S3) → URL stored in DB, delete local copy |
  | Async/long jobs | Queues (RabbitMQ, Kafka) |
- **Uber ride matching:** request hits any server → get location from request → geospatial DB for nearby drivers → match → write to DB → publish Kafka events. Status check later hits a different server, reads DB. NYE ×20 handled by scaling 1,000 → 20,000 instances.
- **Migration changes:** replace in-memory sessions with Redis/JWT; move cart to a DB keyed by user ID (one e-commerce team did it in ~1 week); photo app → upload straight to S3 and store URL.
- **Caching:** shared Redis instead of per-server caches → hit rate ~30% → ~80%.
- **Scheduled jobs:** with 20 identical servers a 9 am email job could run 20 times. Fix: distributed scheduler with locking (Quartz + DB lock), Kubernetes CronJob, or a dedicated job-server tier (Airbnb) while app servers stay stateless.
- **Trade-offs:** slightly higher latency (Redis 1–2 ms vs ns in RAM — negligible next to 10–100 ms network/DB), more network traffic, complexity shifts to supporting services (Redis, queues, S3) — but those are mature.
- **Some state is OK:** WebSocket connections (use reconnect/migration), local ML-model caches, DB connection pools (rebuildable).
- **Principle:** externalise *critical user state*; everything else is negotiable.
- Results cited: LinkedIn 50M → 800M users, Spotify ~500M, Pinterest ~450M users with ~40 deploys/day.

**Remember:** stateless servers + external state stores = the foundation that makes load balancing and scaling actually work.

## 15. Service Discovery
- **Problem:** in microservices, instances keep moving/scaling — how does the user service find the payment service? Hardcoded IPs break.
- **Solution:** a **service registry** ("phone book"). Services register on startup (name + IP), clients query for healthy instances.
- **Two patterns**
  | | Client-side discovery | Server-side discovery |
  |---|---|---|
  | Who looks up | The client queries registry, picks instance (RR/least-conn), calls directly | Client calls an LB/DNS name; LB queries registry and forwards |
  | Example | Netflix **Eureka** (client lib caches locations, refresh ~30 s) | AWS ELB |
  | Pros | Fast (local cache), no extra hop | Simple clients, no library per language |
  | Cons | Needs discovery lib in every service/language | Extra network hop (latency) |
- **Registration styles:** *self-registration* (service registers and sends **heartbeats** every few seconds; no heartbeat → marked unhealthy) vs *third-party registration* (platform does it — Kubernetes watches pods).
- **Tools:** Consul (HashiCorp: registry + health checks + KV + mesh), Eureka (Netflix, JVM, simple/proven), ZooKeeper (Apache, old but reliable, used by Kafka), etcd (distributed KV, backbone of Kubernetes).
- **Kubernetes:** a Service gets a DNS name (e.g. `payment-service`); pods behind it are load-balanced; crashed/unhealthy pods are removed within seconds; new ready pods added automatically — callers see nothing.
- **Enables:** blue-green (flip registry/DNS entry), circuit-breaking (route around unhealthy instances), multi-region failover (nearest healthy region).
- **The registry is critical** → run distributed and replicated. Consul uses **Raft consensus**, 3 or 5 nodes, tolerates minority failure (5 nodes: 2 can fail).
- **Uber scale:** e.g. location ~200 instances, pricing ~150, matching ~300; instances register/deregister automatically as NYE load rises and falls.
- **Challenges:** cache staleness (refresh too often = registry load; too rarely = calls to dead instances); network partition (use last-known data or refuse to call?); registry scaling with thousands of heartbeats (tune intervals); hybrid designs (DNS primary + Consul fallback).
- **Service mesh (Istio):** proxy handles discovery, routing, circuit breaking, retries transparently for app code.

**Remember:** registry + health/heartbeats = dynamic lookup; pick client-side (fast, more code) or server-side (simple, extra hop); protect the registry itself.

## 16. Circuit Breakers & Resilience
- **Cascade failure:** one slow dependency exhausts threads/connection pool in callers, which then fail their callers… whole system down from one external issue.
- **Circuit breaker** (like a house breaker): when a dependency keeps failing, stop calling it and **fail fast**.
- **Three states**
  1. **Closed** — normal; counts successes/failures.
  2. **Open** — threshold exceeded (e.g. 50% failures in last 10, or 5 consecutive): all calls fail immediately in ms; service gets time to recover.
  3. **Half-open** — after timeout (30 s–1 min) let one test request through; success → closed, failure → open again.
- **Numbers:** 1,000 req/s to a DB whose calls now take 10 s instead of 10 ms → without a breaker, 10,000 blocked requests after 10 s and a frozen service; with one, it opens after ~5 failures and the other ~9,995 fail instantly while the service stays responsive.
- **Netflix Hystrix** popularised it (~100B requests/day). Recommendation service down → show popular/trending (**graceful degradation**).
- **Fallbacks:** cached data from 5 min ago; queue payments to process later; browse categories instead of search; generic popular content.
- **Config knobs:** failure threshold, open-state timeout (30 s fast-recovering … 5 min), minimum request volume (e.g. ≥20 req/s) to avoid false trips on low traffic.
- **Related protections:** **bulkheads** (separate breaker per operation, so profile-fetch failing doesn't kill auth), **rate limiting** (e.g. max 100 req/s even when healthy), **aggressive timeouts** (~1 s, not 30 s).
- **Resilience layers:** 1) timeouts on every external call · 2) retries with **exponential backoff** · 3) circuit breakers · 4) bulkheads · 5) fallbacks.
- **Amazon Prime Day:** recommendations down → best sellers; reviews down → hide reviews but keep products + prices.
- **Incident:** slow inventory DB query cascaded through checkout → cart → front-end, site down in 5 min. With breakers: inventory breaker opened, checkout used 5-minute-old cached inventory, ~95% of orders still completed.
- Also acts as automatic rollback safety: bad deploy starts failing → breakers open.
- **Test with chaos** (Netflix Chaos Monkey) — better to find a misconfigured breaker in a drill than in a 2 am outage.
- **Practical tips:** clear error messages ("temporarily unavailable, retry in 30 s", not a bare 500); monitor how often breakers open; tune per business impact (payments get more retries/longer timeouts than recommendations).
- **Libraries:** Spring Boot (Hystrix, **Resilience4j**), Node.js (**opossum**), Python (**pybreaker**), Istio (mesh-level).

**Remember:** fail fast, degrade gracefully; timeouts → retries (backoff) → breaker → bulkhead → fallback.

---

# Cheat sheet — numbers to memorise
- **Time:** 1 day ≈ 86,400 s → use 100,000. Peak = 2–3x average.
- **Sizes:** char 1 B · tweet ~300 B · photo ~4 MB · HD video ~50 MB/min · 4K ~300 MB/min.
- **Units:** KB 10³ · MB 10⁶ · GB 10⁹ · TB 10¹² · PB 10¹⁵. Bandwidth in bits (×8).
- **Availability:** 99% = 3.65 d/yr · 99.9% = 8.76 h · 99.99% = 52 min · 99.999% = ~5 min.
- **Latency targets:** page <2 s · API <200 ms · DB <100 ms.
- **Server capacity:** cached 50–100K QPS · API logic 5–10K · heavy 0.1–1K. Target 60–70% CPU. Add ~20% redundancy + ~10% failure buffer.
- **DB:** ~100–200 connections, ~2K QPS per instance; cache 90% hit cuts DB load 10x.
- **Replication:** ×3. **80/20 rule** for caching.
- **Interview time split:** requirements 10 · high-level 15 · detailed 20 · scale 15.

## Core principles in one list
1. Everything is a trade-off — know which quality attribute wins.
2. Always ask "what if this fails?"
3. Requirements first, specific and measurable.
4. Estimate with orders of magnitude.
5. Scale app tier horizontally (stateless); scale DB vertically as long as you can.
6. Push state out of servers (Redis, DB, S3, queues).
7. Put redundant load balancers in front; health-check everything.
8. Discover services dynamically via a replicated registry.
9. Fail fast with circuit breakers, timeouts, retries, bulkheads, fallbacks.
