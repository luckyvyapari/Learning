# 06 — High Availability & Scalability (ELB + ASG)

## Scalability vs High Availability

| | Meaning | EC2 example |
|---|---|---|
| **Vertical scaling** | Bigger instance (scale up/down) | `t2.nano` → `u-12tb1.metal`. Common for DBs (RDS, ElastiCache). Hardware limit |
| **Horizontal scaling** (elasticity) | More instances (scale out/in) | ASG + Load Balancer |
| **High availability** | Run in **≥ 2 AZs** to survive data center loss | Multi-AZ ASG + multi-AZ LB (can be passive e.g. RDS Multi-AZ, or active) |

## Load Balancer (ELB)

Forwards traffic to multiple downstream servers.

Why: spread load · single DNS entry point · handle instance failures · health checks · **SSL termination** · stickiness · multi-AZ · separate public/private traffic.

Managed by AWS (upgrades, HA, maintenance). Integrates with EC2, ASG, ECS, ACM, CloudWatch, Route 53, WAF, Global Accelerator.

**Health checks**: on port + route (e.g. `/health`); response must be **200 OK** else unhealthy.

**Security groups pattern**: LB SG allows HTTP/HTTPS from anywhere; EC2 SG allows traffic **only from the LB's SG**.

## Types of Load Balancer

| Type | Year | Layer | Protocols | Notes |
|---|---|---|---|---|
| **CLB** | 2009 | 4 & 7 | HTTP, HTTPS, TCP, SSL | Legacy, one SSL cert only |
| **ALB** | 2016 | **7** | HTTP, HTTPS, WebSocket, HTTP/2 | Content-based routing |
| **NLB** | 2017 | **4** | TCP, TLS, UDP | Ultra-low latency, millions rps |
| **GWLB** | 2020 | **3** | IP packets (GENEVE, port 6081) | 3rd-party virtual appliances |

### ALB
- Routes to **target groups** by: **URL path**, **hostname**, **query string**, **headers**
- Redirects (HTTP→HTTPS), multiple apps per machine, port mapping for ECS dynamic ports
- Great for microservices/containers
- **Target groups**: EC2 (ASG), ECS tasks, **Lambda** (request → JSON event), **IP addresses (private only)**
- Health checks at target group level
- Fixed hostname; client IP is in **`X-Forwarded-For`** (also `X-Forwarded-Port`, `X-Forwarded-Proto`) — app doesn't see client IP directly

### NLB
- **One static IP per AZ**, supports **Elastic IP** → good for IP whitelisting
- Extreme performance, TCP/UDP
- Targets: EC2, private IPs, **ALB**
- Health checks: TCP, HTTP, HTTPS

### GWLB
- Deploy/scale/manage 3rd-party **network appliances**: firewalls, IDS/IPS, deep packet inspection
- Transparent network gateway (single entry/exit) + load balancer
- Targets: EC2, private IPs

## Sticky Sessions (Session Affinity)

Same client → same instance. Works on **CLB, ALB, NLB**. Use: keep session data. Downside: can unbalance load.

| Cookie type | Name |
|---|---|
| Application-based, custom | Set per target group; **don't use** `AWSALB`, `AWSALBAPP`, `AWSALBTG` |
| Application-based, LB-generated | `AWSALBAPP` |
| Duration-based (LB-generated) | `AWSALB` (ALB), `AWSELB` (CLB) |

## Cross-Zone Load Balancing

Each LB node distributes evenly across **all instances in all AZs** (instead of only its own AZ).

| LB | Default | Inter-AZ charge |
|---|---|---|
| ALB | **Enabled** (disable at target group) | Free |
| NLB / GWLB | Disabled | **Pay** if enabled |
| CLB | Disabled | Free if enabled |

## SSL/TLS

- Encrypts traffic client ↔ LB (in-flight). TLS is the modern SSL
- LB uses an **X.509 certificate**; manage with **ACM** or upload your own
- HTTPS listener: must set a default cert; optional extra certs for multiple domains
- **SNI (Server Name Indication)**: client sends hostname in handshake so server picks correct cert → multiple certs on one listener
- **SNI works on ALB, NLB, CloudFront. NOT on CLB**
- Security policy can allow legacy SSL/TLS versions

| LB | Certs |
|---|---|
| CLB | One cert (multiple hostnames need multiple CLBs) |
| ALB / NLB | Multiple certs via SNI |

## Connection Draining / Deregistration Delay

- Name: **Connection Draining** (CLB), **Deregistration Delay** (ALB/NLB)
- Time to finish in-flight requests while instance is deregistering/unhealthy; stops sending new requests
- 1–3600 s, **default 300 s**, 0 = disabled; set low for short requests

## Auto Scaling Group (ASG)

Scale out/in to match load · enforce min/max · auto-register to LB · replace unhealthy instances. **ASG is free** (pay for EC2).

**Capacity**: Minimum ≤ Desired ≤ Maximum.

**Launch Template** (replaces Launch Configurations): AMI + instance type, user data, EBS, security groups, key pair, IAM role, VPC/subnets, LB info.

### Scaling Policies

| Policy | How |
|---|---|
| **Target tracking** | Simplest — keep a metric at target (e.g. avg CPU ≈ 40%) |
| **Simple / Step** | CloudWatch alarm → add/remove N units (CPU > 70% add 2, < 30% remove 1) |
| **Scheduled** | Known patterns (min 10 at 5pm Fridays) |
| **Predictive** | Forecast load, schedule ahead |

Good metrics: `CPUUtilization`, **`RequestCountPerTarget`**, avg Network In/Out, custom CloudWatch metrics.

**Cooldown** (default **300 s**): after a scaling action, no further launch/terminate — lets metrics stabilize. Use ready-made AMI to shorten.

ASG health can use ELB health checks (not just EC2 status).

## Exam Hints

- Static IP / extreme perf / UDP → **NLB**
- Path/host routing, microservices, Lambda target → **ALB**
- Firewall/IDS appliance fleet → **GWLB**
- Client IP lost behind ALB → `X-Forwarded-For`
- Multiple SSL certs → ALB/NLB + SNI
- Scale on request load → target tracking on `RequestCountPerTarget`
