# 09 — Classic Solutions Architectures

How the pieces fit together. Know this section **cold** — exam questions are scenario-based.

## Case 1 — Stateless web app (WhatIsTheTime.com)

No database, start small, scale without downtime. Journey:

| Step | Design | Problem / lesson |
|---|---|---|
| 1 | One public EC2 + **Elastic IP** | Simple |
| 2 | Scale **vertically** (t2 → m5) | **Downtime** while upgrading |
| 3 | Scale **horizontally**, multiple EC2 with **Route 53 A record** (TTL 1h), no Elastic IP | Clients cache DNS → they hit removed instances; no health checks |
| 4 | **ELB + health checks**, private EC2, Route 53 **Alias** | Instances hidden behind LB; SG: EC2 allows only LB |
| 5 | Add **Auto Scaling Group** | Auto add/remove/replace instances |
| 6 | **Multi-AZ** ASG + ELB | Survive AZ disaster |
| 7 | **Reserved Instances/Savings Plan** for minimum capacity | Cost savings |

Concepts: public vs private IP · Elastic IP vs Route 53 vs ELB · TTL, A vs Alias · ASG vs manual · Multi-AZ · health checks · SG rules · reserved capacity.

## Case 2 — Stateful web app (MyClothes.com)

Hundreds of concurrent users, shopping cart must survive, user details in a DB, keep app **as stateless as possible**.

| Step | Technique | Trade-off |
|---|---|---|
| Sticky sessions | **ELB stickiness** | User may lose cart if instance dies; load imbalance |
| **User cookies** | Cart stored client-side (stateless web) | Heavier HTTP requests, **security risk** (tamperable → validate), **< 4 KB** |
| **Server session** | Cookie holds only `session_id`; session data in **ElastiCache** (alt: **DynamoDB**) | Best pattern |
| User data | **RDS** (address, name…) | — |
| Scale reads | **RDS Read Replicas**, or **ElastiCache lazy loading** cache | — |
| Survive disaster | **Multi-AZ** for ElastiCache + RDS | — |
| Security | Tiered SGs: LB open to `0.0.0.0/0` (HTTP/HTTPS) → EC2 SG allows only LB SG → ElastiCache & RDS SGs allow only EC2 SG | SGs reference each other |

**3-tier**: Route 53 → ELB (public subnet) → ASG EC2 (private subnet) → ElastiCache/RDS (data subnet).

## Case 3 — MyWordPress.com

Scalable WordPress; images must display from any instance; content in MySQL.

- **DB**: RDS MySQL Multi-AZ, or **Aurora MySQL** (easy Multi-AZ + Read Replicas)
- **Images** options:
  - **EBS**: one volume per instance — fine for **single instance**; with many instances images are on only one volume ❌
  - **EFS** (via ENIs in each AZ): shared network file system — correct for **distributed** apps ✅

## Instantiating Applications Quickly

Full stack boot is slow (install, load data, configure). Speed up:

| Resource | Technique |
|---|---|
| EC2 | **Golden AMI** (pre-installed), **User Data** (dynamic config), or hybrid (Elastic Beanstalk) |
| RDS | **Restore from snapshot** (schema + data ready) |
| EBS | **Restore from snapshot** (formatted, has data) |

## Elastic Beanstalk

Developer-centric PaaS: you provide **code**, it handles capacity provisioning, load balancing, scaling, health monitoring, instance config. Uses EC2, ASG, ELB, RDS underneath. **Free — pay only for underlying resources.** You keep full config control.

| Component | Meaning |
|---|---|
| **Application** | Collection of environments, versions, configs |
| **Application Version** | Iteration of app code |
| **Environment** | Resources running **one** version at a time; create many (dev/test/prod) |
| **Tiers** | **Web Server tier** (ELB + EC2) and **Worker tier** (EC2 pulling from **SQS**, scale on queue length) |

**Platforms**: Go, Java SE, Java Tomcat, .NET Core on Linux, .NET on Windows, Node.js, PHP, Python, Ruby, Packer Builder, Docker (single, multi-container, preconfigured).

**Deployment modes**: **Single Instance** (Elastic IP, great for dev) vs **High Availability with Load Balancer** (ALB + ASG multi-AZ + RDS Multi-AZ, great for prod).

## Exam Hints

- Stateless + session → store session in **ElastiCache/DynamoDB**, cookie = session id
- Shared files across instances → **EFS** (not EBS)
- Developer wants "just upload code" → **Elastic Beanstalk**
- Fast recovery/launch → Golden AMI + snapshots
- Hide instances → private subnets + ELB + SG chaining
