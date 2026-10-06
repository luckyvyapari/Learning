# 08 — Amazon Route 53

## DNS Basics

DNS translates hostnames → IPs. Hierarchy: Root → TLD (`.com`) → SLD (`example.com`) → subdomain. FQDN = full name (`api.www.example.com.`).

Terms: **Domain Registrar**, **DNS records**, **Zone file**, **Name server** (authoritative or not), TLD, SLD.

## Route 53

- Highly available, scalable, managed, **authoritative DNS** (you control records)
- Also a **Domain Registrar**
- Health checks on resources
- **Only AWS service with 100% availability SLA**
- "53" = traditional DNS port

## Records

Each record: **name · type · value · routing policy · TTL**.

| Type | Maps to |
|---|---|
| **A** | Hostname → IPv4 |
| **AAAA** | Hostname → IPv6 |
| **CNAME** | Hostname → another hostname (target must resolve via A/AAAA). **Not allowed on zone apex** (`example.com`) |
| **NS** | Name servers of the hosted zone |

(Advanced: CAA, DS, MX, NAPTR, PTR, SOA, TXT, SPF, SRV.)

### Hosted Zones ($0.50/month each)
- **Public**: route traffic on the internet
- **Private**: route traffic **within one or more VPCs** (e.g. `db.example.internal`)

### TTL
- High TTL (24h): less Route 53 traffic, may serve outdated records
- Low TTL (60s): more traffic/cost, easy changes
- **TTL mandatory for every record except Alias**

### CNAME vs Alias

| | CNAME | Alias |
|---|---|---|
| Target | Any hostname | **AWS resource** |
| Root domain | **No** | **Yes** |
| Cost | Charged | **Free** |
| Health check | No | **Native** |
| TTL | Set | **Can't set** |
| Record type | CNAME | **A/AAAA** |

**Alias targets**: ELB, CloudFront, API Gateway, Elastic Beanstalk, S3 websites, VPC interface endpoints, Global Accelerator, another record in same hosted zone. **Cannot alias an EC2 DNS name.**

## Routing Policies

DNS only **answers queries** — it does not route traffic like a load balancer.

| Policy | Behavior | Health checks |
|---|---|---|
| **Simple** | One or multiple values; client picks random one; alias = one resource | **No** |
| **Weighted** | % by weight (no need to sum to 100); same name+type; weight 0 stops traffic (all 0 = equal) | Yes |
| **Latency-based** | Lowest latency Region for user | Yes |
| **Failover** | Active-passive primary/secondary | **Mandatory** |
| **Geolocation** | By user **location** (continent/country/US state); create a **Default** record | Yes |
| **Geoproximity** | By geographic location of users **and resources**, with **bias** (+1..99 expand, −1..−99 shrink); needs **Traffic Flow**; AWS or non-AWS (lat/long) | — |
| **IP-based** | By client CIDR → endpoint mapping (CIDR collections) | — |
| **Multi-value** | Returns up to **8 healthy** records; **not a substitute for ELB** | Yes |

Weighted use: test new versions, balance across regions. Geolocation use: localization, content restriction.

## Health Checks

Types:
1. **Endpoint** health checks (public resources)
2. **Calculated** health checks (combine children with OR/AND/NOT; up to **255** children) — maintenance without failing everything
3. **CloudWatch Alarm** health checks — for **private resources** (Route 53 checkers are outside the VPC)

Endpoint details: ~15 global checkers · healthy/unhealthy threshold **3** · interval **30 s** (10 s costs more) · HTTP/HTTPS/TCP · healthy if **> 18%** of checkers say healthy · pass on **2xx/3xx** · can check text in first **5120 bytes** · must allow Route 53 checker IP ranges.

## Registrar vs DNS Service

- Registrar = where you buy the domain (GoDaddy, Amazon Registrar)
- DNS service = where records live
- Buy at 3rd party, use Route 53: **create Hosted Zone → update NS records at registrar to Route 53 name servers**

## Hybrid DNS (Route 53 Resolver)

Resolver answers by default for EC2 local names, private hosted zones, public name servers. For hybrid with on-premises (Direct Connect / VPN):
- **Inbound endpoint**: on-prem resolvers → resolve AWS names
- **Outbound endpoint**: Route 53 Resolver → forwards queries to on-prem resolvers

## Exam Hints

- Root domain → ELB/CloudFront → **Alias A record**
- Region failover → **Failover** policy with health check
- Different content per country → **Geolocation**; fastest region → **Latency**
- Gradual rollout 90/10 → **Weighted**
- Private resource health → CloudWatch alarm health check
