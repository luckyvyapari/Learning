# 13 — CloudFront & Global Accelerator

## CloudFront (CDN)

Content cached at **edge locations** (hundreds of PoPs) → better read performance, lower latency. Built-in **DDoS protection**, integrates with **AWS Shield** and **WAF**.

### Origins

| Origin | Notes |
|---|---|
| **S3 bucket** | Distribute + cache files; also upload via CloudFront. Secured with **Origin Access Control (OAC)** + S3 bucket policy |
| **VPC Origin** | Apps in **private subnets**: private ALB, NLB, EC2 — no need to expose to internet |
| **Custom origin (HTTP)** | S3 website (enable static hosting first), public ALB, any public HTTP backend |

**Public ALB/EC2 as origin**: EC2 security group must allow the **public IPs of edge locations** (published list); ALB must be public, EC2 behind it can be private (SG allows ALB SG).

### CloudFront vs S3 Cross-Region Replication

| | CloudFront | S3 CRR |
|---|---|---|
| Network | Global edge | Set up per target region |
| Freshness | Cached for a **TTL** (maybe a day) | **Near real-time** |
| Access | — | **Read only** |
| Best for | **Static content available everywhere** | **Dynamic content** needing low latency in a few regions |

### Geo Restriction
- **Allowlist** (only listed countries) or **Blocklist** (banned countries)
- Country determined via 3rd-party Geo-IP database. Use: copyright/licensing

### Cache Invalidation
CloudFront only refreshes after TTL expires. Force refresh with **invalidation**: all files (`*`) or a path (`/images/*`).

## Global Accelerator

Problem: global users go over the public internet (many hops, latency).

- Uses the **AWS internal network** to reach your app
- **2 static Anycast IPs** created; traffic enters nearest edge location, then travels private AWS backbone to the app
- **Unicast** = one server one IP; **Anycast** = all servers same IP, client routed to nearest

Features:
- Works with **Elastic IP, EC2, ALB, NLB** (public or private)
- Consistent performance: lowest-latency routing, **fast regional failover**
- No client DNS cache issue (IPs don't change)
- **Health checks** → failover **< 1 min**; great for DR
- Security: only **2 IPs to whitelist**; **Shield** DDoS protection

## Global Accelerator vs CloudFront

| | CloudFront | Global Accelerator |
|---|---|---|
| Common | AWS edge network + global backbone, **Shield** integration | same |
| Content | **Cacheable** (images, video) **and dynamic** (API acceleration) served **at the edge** | **Proxies packets** to apps in one or more regions — no caching |
| Protocols | HTTP | **TCP or UDP** |
| Good for | Static/dynamic web content | **Gaming (UDP), IoT (MQTT), VoIP**; HTTP needing **static IPs**; HTTP needing **deterministic fast regional failover** |

## Exam Hints

- Cache static content globally, with WAF/Shield → **CloudFront**
- Restrict content by country → **CloudFront geo restriction**
- Private S3 behind CloudFront → **OAC**
- Static IPs + non-HTTP + fast failover → **Global Accelerator**
- Stale cached file → **invalidation**
