# AWS Solutions Architect Associate (SAA-C03) — Complete Quick Revision

One file covering all topics. Each topic has the key facts, then **exam-style practice questions** (click "Answer" to reveal). Full detail lives in the numbered files (01–29).

---

## Official AWS Resources (start here)

| What | Link | Use it for |
|---|---|---|
| **Exam page** (exam guide, sample questions) | https://aws.amazon.com/certification/certified-solutions-architect-associate/ | Official exam guide, domains, sample questions, booking |
| **AWS Skill Builder** | https://skillbuilder.aws/ | **Official practice question set** + free exam prep courses |
| **AWS Architecture Center** | https://aws.amazon.com/architecture/ | **Real architecture examples**, best practices, patterns |
| **Reference Architecture Diagrams** | https://aws.amazon.com/architecture/reference-architecture-diagrams/ | Ready-made diagrams by industry and use case |
| **This is My Architecture** (videos) | https://aws.amazon.com/architecture/this-is-my-architecture/ | Real companies explaining their AWS architecture |
| **AWS Solutions Library** | https://aws.amazon.com/solutions/ | Vetted deployable solutions (CloudFormation) |
| **Amazon Builders' Library** | https://aws.amazon.com/builders-library/ | How Amazon itself builds and operates systems |
| **Well-Architected Framework** | https://aws.amazon.com/architecture/well-architected/ | 6 pillars + lenses |
| **Well-Architected Tool** | https://console.aws.amazon.com/wellarchitected | Review your own workload against the pillars |
| **Whitepapers** | https://aws.amazon.com/whitepapers/ | Architecting for the Cloud, Well-Architected, DR |
| **Disaster Recovery** | https://aws.amazon.com/disaster-recovery/ | DR strategies (RPO/RTO) |
| **AWS FAQs** | https://aws.amazon.com/faqs/ | Per-service FAQs — many exam questions come from here (e.g. https://aws.amazon.com/vpc/faqs/) |
| **AWS Workshops** | https://workshops.aws/ | Free hands-on labs |
| **AWS Samples (GitHub)** | https://github.com/aws-samples | Example code + architectures |
| **AWS re:Post** | https://repost.aws/ | Community Q&A |
| **Global Infrastructure** | https://infrastructure.aws/ | Regions / AZs / edge map |
| **Pricing Calculator** | https://calculator.aws/ | Estimate cost of an architecture |
| **IAM policy evaluation logic** | https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html | Allow/Deny decision flow |
| **Architecture Icons** | https://aws.amazon.com/architecture/icons/ | Draw your own diagrams |

---

## Exam at a Glance

| Item | Detail |
|---|---|
| Questions / time | **65 questions, 130 minutes** (multiple choice + multiple response) |
| Pass | **720 / 1000** |
| Fee | 150 USD · retake after 14 days |
| Style | Scenario-based. Eliminate wrong answers. "Feasible but highly complicated" is usually wrong |

| Domain | Weight |
|---|---|
| 1. Design **Secure** Architectures | 30% |
| 2. Design **Resilient** Architectures | 26% |
| 3. Design **High-Performing** Architectures | 24% |
| 4. Design **Cost-Optimized** Architectures | 20% |

**Keyword → answer reflexes**: "least operational overhead" → managed/serverless · "most cost-effective" → Spot / S3 IA / Glacier / Savings Plans · "highly available" → Multi-AZ · "decouple" → SQS/SNS · "real-time" → Kinesis · "static IP" → NLB / Global Accelerator · "audit API calls" → CloudTrail · "compliance of config" → Config.

---

## 01. Global Infrastructure

- **Region** = cluster of data centers; **AZ** = 1+ isolated data centers (3 typical, min 3, max 6); **Edge locations** 400+ (CloudFront, Route 53, Global Accelerator)
- Choose region by: **compliance**, **latency**, **service availability**, **pricing**
- **Global** services: IAM, Route 53, CloudFront, WAF. Most others are **regional**

**Q1.** A company must keep customer data inside Germany by law. What decides the Region choice first?
A) Lowest price B) Compliance / data governance C) Number of AZs D) Newest services

<details><summary>Answer</summary>

**B** — Data never leaves a region without your permission; compliance comes before price or latency.

</details>

---

## 02. IAM

- **Users** (1 per person) · **Groups** (users only, no nested groups) · **Policies** (JSON: Effect, Action, Resource, Condition, Principal) · **Roles** (for AWS services)
- **Least privilege**. Root account only for account setup. Enforce **MFA** + password policy
- Console = password + MFA; CLI/SDK = **access keys** (never share, never put on EC2)
- **Credentials Report** (account level, all users' credentials) · **Access Advisor** (user level, services used + last accessed)

**Q2.** An application on EC2 must read from S3. What is the most secure way to give access?
A) Store access keys in the AMI B) Store keys in user data C) Attach an IAM role to the instance D) Make the bucket public

<details><summary>Answer</summary>

**C** — IAM roles give temporary credentials automatically; never embed access keys.

</details>

**Q3.** Security wants to remove permissions users never use. Which tool shows services a user accessed and when?
A) Credentials Report B) Access Advisor C) CloudTrail Insights D) Trusted Advisor

<details><summary>Answer</summary>

**B** — IAM Access Advisor is user-level "last accessed" data. Credentials Report lists credential status.

</details>

---

## 03. EC2 Basics

- Type naming `m5.2xlarge` = class · generation · size. **General (t, m)**, **Compute (c)**, **Memory (r, x)**, **Storage (i, d)**
- **User Data** runs once at first boot, as root
- **Security Groups**: allow rules only, stateful, inbound blocked / outbound allowed by default, can reference other SGs. **Timeout = SG issue; connection refused = app issue**
- Ports: 22 SSH/SFTP, 21 FTP, 80 HTTP, 443 HTTPS, 3389 RDP

| Purchase option | Discount | Use |
|---|---|---|
| On-Demand | 0 | Short, unpredictable |
| Reserved (1/3 yr) | up to 72% | Steady (databases) |
| Convertible RI | up to 66% | Flexible family/OS |
| Savings Plan | up to 72% | $/hour commitment |
| **Spot** | up to **90%** | Fault-tolerant batch; 2-min warning |
| Dedicated Host | — | **BYOL**, compliance |
| Dedicated Instance | — | Hardware not shared with other customers |
| Capacity Reservation | 0 | Guaranteed capacity in an AZ |

- Spot: **cancel Spot request first, then terminate** instances. Spot Fleet strategy `priceCapacityOptimized` recommended

**Q4.** A nightly image-processing job can be interrupted and restarted. Cheapest option?
A) On-Demand B) Reserved C) Spot D) Dedicated Host

<details><summary>Answer</summary>

**C** — Interruptible, fault-tolerant work = Spot (up to 90% off).

</details>

**Q5.** Users get a **timeout** reaching a web server on EC2. Most likely cause?
A) App crashed B) Security group doesn't allow the port C) Wrong AMI D) EBS full

<details><summary>Answer</summary>

**B** — Timeout = security group / network. "Connection refused" would mean the app.

</details>

**Q6.** Software licensed per physical CPU socket must move to AWS. Which option?
A) Dedicated Instances B) Dedicated Hosts C) Spot D) Savings Plan

<details><summary>Answer</summary>

**B** — Dedicated Hosts give visibility of sockets/cores for BYOL licensing.

</details>

---

## 04. EC2 Associate

- Public IP changes on stop/start → **Elastic IP** fixed (max 5/account; prefer DNS or LB)
- **Placement groups**: **Cluster** (same AZ, low latency, 10 Gbps, HPC) · **Spread** (separate hardware, max **7 per AZ**, critical apps) · **Partition** (racks, 100s of instances — Hadoop, Cassandra, Kafka)
- **ENI**: virtual NIC, AZ-bound, move between instances for failover
- **Hibernate**: RAM saved to **encrypted root EBS**, RAM < 150 GB, max 60 days → fast boot

**Q7.** A big data job needs the lowest network latency between instances. Which placement group?
A) Spread B) Partition C) Cluster D) None

<details><summary>Answer</summary>

**C** — Cluster puts instances close together in one AZ (but whole group fails with the AZ).

</details>

**Q8.** An app takes 10 minutes to warm in-memory caches after start. How to resume faster after stop?
A) Use instance store B) EC2 Hibernate C) Larger instance D) Placement group

<details><summary>Answer</summary>

**B** — Hibernate keeps RAM state on the encrypted root EBS volume, so the OS and caches resume instead of rebooting.

</details>

---

## 05. EC2 Storage

- **EBS**: network drive, **one AZ**, one instance (except io1/io2 **Multi-Attach** up to 16, same AZ). Move AZ → **snapshot + restore**. Root deleted on terminate by default
- Snapshots: copy cross-AZ/region, **Archive** (75% cheaper, 24–72 h restore), **Recycle Bin**, **Fast Snapshot Restore**
- **AMI**: region-specific, copyable; golden image for fast boot
- **Instance Store**: highest IOPS, **ephemeral**

| EBS type | Key fact |
|---|---|
| gp3 | 3,000 IOPS baseline, up to 80,000; IOPS independent of size |
| gp2 | 3 IOPS/GB, max 16,000 |
| io1 / io2 Block Express | Provisioned IOPS, io2 up to **256,000**, Multi-Attach |
| st1 / sc1 | HDD, throughput / cold; **cannot boot** |

- Encrypt existing volume: **snapshot → copy with encryption → new volume**
- **EFS**: managed NFS, **Linux only**, multi-AZ, auto-scales, pay per use; modes General Purpose / Max I/O; throughput Bursting/Provisioned/Elastic; tiers Standard / IA / Archive

**Q9.** Many EC2 Linux instances across 3 AZs need the same shared files. Which storage?
A) EBS gp3 B) Instance Store C) EFS D) EBS Multi-Attach

<details><summary>Answer</summary>

**C** — EFS is multi-AZ shared NFS. Multi-Attach is single-AZ and io1/io2 only.

</details>

**Q10.** A database needs 50,000 sustained IOPS on EBS with the highest durability. Choose:
A) gp2 B) st1 C) io2 Block Express D) sc1

<details><summary>Answer</summary>

**C** — Provisioned IOPS SSD for critical DBs. (gp3 can reach 80k, but io2 is the "sustained, critical" answer.)

</details>

**Q11.** How do you move an EBS volume from us-east-1a to us-east-1b?
A) Detach and attach B) Snapshot then restore in 1b C) Enable Multi-Attach D) Use EFS

<details><summary>Answer</summary>

**B** — EBS is AZ-locked; snapshots are the way to move.

</details>

---

## 06. ELB + ASG

| LB | Layer | Pick when |
|---|---|---|
| **ALB** | 7 | Path/host/query/header routing, microservices, Lambda targets, redirects |
| **NLB** | 4 | **Static IP per AZ / Elastic IP**, TCP/UDP, extreme performance |
| **GWLB** | 3 | Fleet of firewalls/IDS (GENEVE 6081) |
| CLB | 4/7 | Legacy |

- Client IP behind ALB → **X-Forwarded-For**
- **Sticky sessions** (cookies AWSALB/AWSALBAPP) · **Cross-zone**: ALB on & free; NLB/GWLB off & paid
- **SNI** = multiple SSL certs on one listener (**ALB, NLB, CloudFront — not CLB**); certs from **ACM**
- **Deregistration delay** default 300 s
- **ASG**: min/desired/max, Launch Template, replaces unhealthy instances, free
- Scaling: **Target tracking** (simplest), Step/Simple, **Scheduled**, **Predictive**. Good metric: `RequestCountPerTarget`. Cooldown 300 s
- SG pattern: EC2 allows traffic only from the **LB's security group**

**Q12.** A partner must whitelist a fixed IP for a TCP service. Which LB?
A) ALB B) NLB with Elastic IP C) CLB D) GWLB

<details><summary>Answer</summary>

**B** — NLB has static IPs per AZ and supports Elastic IPs.

</details>

**Q13.** `/api/*` must go to one target group and `/images/*` to another. Which LB?
A) NLB B) ALB C) GWLB D) Route 53

<details><summary>Answer</summary>

**B** — Path-based routing is an ALB (layer 7) feature.

</details>

**Q14.** The app behind an ALB logs the ALB's IP instead of the user's. Fix?
A) Use NLB B) Read the X-Forwarded-For header C) Enable stickiness D) Use Elastic IP

<details><summary>Answer</summary>

**B** — ALB terminates the connection; the real client IP is in X-Forwarded-For.

</details>

**Q15.** Traffic rises every Monday 9 AM. Simplest ASG approach?
A) Target tracking only B) Scheduled scaling C) Manual D) Bigger instances

<details><summary>Answer</summary>

**B** — Known, predictable pattern = scheduled scaling (predictive scaling is also valid when patterns are learned).

</details>

---

## 07. RDS, Aurora, ElastiCache

| | Read Replicas | Multi-AZ |
|---|---|---|
| Purpose | Scale **reads** | **HA / DR** |
| Replication | **Async** | **Sync** |
| Count | up to 15 | 1 standby (not readable) |
| Failover | Manual promote | **Automatic**, same DNS |

- RDS: storage auto scaling, PITR backups 1–35 days, **no SSH** (except **RDS Custom** for Oracle/SQL Server), encryption at launch (snapshot → restore encrypted), IAM auth
- **RDS Proxy**: connection pooling, Lambda-friendly, failover −66%, VPC only, Secrets Manager
- **Aurora**: MySQL/Postgres compatible, **6 copies in 3 AZ**, up to 15 replicas, auto-grows to 256 TB, failover < 30 s, writer + reader + custom endpoints, **Serverless**, **Global Database** (< 1 s lag, RTO < 1 min), **Backtrack**, **Cloning**, **Babelfish** (T-SQL)
- **ElastiCache**: Redis (HA, persistence, sorted sets → leaderboards) vs Memcached (sharding, multi-thread, no HA). Patterns: **lazy loading** (may be stale), **write-through** (never stale), **session store** (TTL). Needs code changes

**Q16.** A reporting job slows the production MySQL database. Best fix?
A) Multi-AZ B) Read Replica for reporting C) Bigger instance D) ElastiCache

<details><summary>Answer</summary>

**B** — Offload reads to a replica. Multi-AZ standby can't serve reads.

</details>

**Q17.** Thousands of Lambda invocations exhaust RDS connections. Solution?
A) Increase instance size B) RDS Proxy C) Multi-AZ D) DynamoDB

<details><summary>Answer</summary>

**B** — RDS Proxy pools and shares connections (Lambda must be in the VPC).

</details>

**Q18.** Global app needs a relational DB with low-latency reads in 3 regions and DR RTO under 1 minute.
A) RDS cross-region replicas B) Aurora Global Database C) DynamoDB Global Tables D) Multi-AZ

<details><summary>Answer</summary>

**B** — Aurora Global: < 1 s replication, promote a secondary region in < 1 min. (DynamoDB is NoSQL.)

</details>

**Q19.** You need a real-time gaming leaderboard. Which cache?
A) Memcached B) ElastiCache for Redis (Sorted Sets) C) DAX D) CloudFront

<details><summary>Answer</summary>

**B** — Redis sorted sets keep members unique and ordered in real time.

</details>

---

## 08. Route 53

- Authoritative DNS + registrar, **100% SLA**. Records: **A** (IPv4), **AAAA** (IPv6), **CNAME** (not at zone apex), **NS**
- **Alias**: to AWS resources (ELB, CloudFront, API Gateway, S3 website, Beanstalk, Global Accelerator, VPC interface endpoint), **works at apex**, free, health-checked, no TTL. **Not to EC2 DNS name**
- Hosted zones: public / private (VPC). $0.50/month
- Routing: **Simple** (no health check) · **Weighted** · **Latency** · **Failover** (health check mandatory) · **Geolocation** (user location, add Default) · **Geoproximity** (bias, Traffic Flow) · **IP-based** (CIDR) · **Multi-value** (up to 8 healthy, not an ELB)
- Health checks: endpoint (15 global checkers, > 18% healthy, 2xx/3xx), **calculated** (AND/OR/NOT, 255 children), **CloudWatch alarm** (for private resources)
- 3rd-party registrar → create hosted zone, update **NS records** at registrar
- **Resolver endpoints**: **Inbound** (on-prem resolves AWS names), **Outbound** (AWS forwards to on-prem)

**Q20.** Point `example.com` (zone apex) to an ALB.
A) CNAME B) A record with ALB IP C) Alias record D) MX record

<details><summary>Answer</summary>

**C** — CNAME is not allowed at the apex; ALB IPs change; Alias handles it for free.

</details>

**Q21.** Send 10% of users to a new app version for testing.
A) Latency B) Weighted C) Geolocation D) Failover

<details><summary>Answer</summary>

**B** — Weighted routing (e.g. 90/10).

</details>

**Q22.** French users must see the French site regardless of latency.
A) Latency B) Geoproximity C) Geolocation D) Simple

<details><summary>Answer</summary>

**C** — Geolocation routes by user location (country/continent).

</details>

**Q23.** Route 53 must fail over based on a private database's health (inside a VPC).
A) HTTP health check on private IP B) Health check on a CloudWatch alarm C) Calculated check D) Not possible

<details><summary>Answer</summary>

**B** — Route 53 checkers live outside the VPC; monitor a CloudWatch metric/alarm instead.

</details>

---

## 09. Classic Architectures

- **Stateless app journey**: single EC2 + EIP → vertical scale (downtime) → many EC2 + DNS A records (TTL problems) → **ELB + health checks** + Alias → **ASG** → **Multi-AZ** → **reserve minimum capacity**
- **Stateful app (shopping cart)**: stickiness (lose on instance failure) → cookies (< 4 KB, tamperable) → **session id in cookie + session in ElastiCache/DynamoDB** (best). User data in **RDS**, scale reads with **replicas** or **cache**, **Multi-AZ** everything, SG chaining (LB → EC2 → DB)
- **WordPress**: Aurora MySQL; shared images on **EFS** (EBS only works for single instance)
- **Fast instantiation**: **Golden AMI** + **User Data** (dynamic config); RDS/EBS **restore from snapshot**
- **Elastic Beanstalk**: PaaS for developers (code only), uses EC2/ASG/ELB/RDS, free (pay for resources). **Web tier** vs **Worker tier** (SQS). Modes: single instance (dev) / HA with LB (prod)

**Q24.** Users lose their shopping cart when an instance is replaced by the ASG. Best stateless fix?
A) Sticky sessions B) Store cart in cookies C) Store session data in ElastiCache keyed by session ID D) Bigger instances

<details><summary>Answer</summary>

**C** — Externalize session state; stickiness still loses data when an instance dies.

</details>

**Q25.** A developer team wants to upload code and let AWS handle capacity, load balancing and scaling.
A) CloudFormation B) Elastic Beanstalk C) EC2 + ASG manually D) Lambda@Edge

<details><summary>Answer</summary>

**B** — Elastic Beanstalk is the developer-centric managed platform.

</details>

---

## 10. S3 Core

- Buckets are **regional**, names **globally unique**; object key = full path (no real folders); max object **50 TB**, **multi-part required > 5 GB**
- Access if (IAM allow **OR** resource policy allow) **AND no explicit deny**. **Bucket policy** for public access, forced encryption, **cross-account**. **Block Public Access** (account level)
- Static website: **403** → bucket policy doesn't allow public read
- **Versioning** (bucket level; pre-existing objects = `null`; suspending keeps versions)
- **Replication (CRR / SRR)**: versioning on both sides, async, **only new objects** (existing → **Batch Replication**), delete markers optional, version-ID deletes not replicated, **no chaining**
- Durability **11 9s** for all classes

| Class | Min days | Note |
|---|---|---|
| Standard | — | 99.99% availability |
| Standard-IA | 30 | Backups, DR |
| One Zone-IA | 30 | Re-creatable data, single AZ |
| Glacier Instant | 90 | ms retrieval, quarterly access |
| Glacier Flexible | 90 | 1–5 min / 3–5 h / 5–12 h |
| Glacier Deep Archive | 180 | 12 h / 48 h, cheapest |
| Intelligent-Tiering | — | Auto-moves, monitoring fee, **no retrieval fee** |
| Express One Zone | — | Directory bucket, single-digit ms, 10x faster |

**Q26.** Static website on S3 returns 403 Forbidden.
A) Enable versioning B) Add bucket policy allowing public s3:GetObject (and disable Block Public Access) C) Enable CORS D) Use SSE-KMS

<details><summary>Answer</summary>

**B** — Website hosting needs public read permission.

</details>

**Q27.** Thumbnails can be regenerated anytime and are rarely accessed. Cheapest suitable class?
A) Standard B) Standard-IA C) One Zone-IA D) Glacier Deep Archive

<details><summary>Answer</summary>

**C** — Re-creatable + infrequent → One Zone-IA. Deep Archive retrieval is too slow for serving.

</details>

**Q28.** Replication was enabled but old objects didn't copy. Fix?
A) Disable versioning B) S3 Batch Replication C) Enable SRR D) Lifecycle rule

<details><summary>Answer</summary>

**B** — Replication only applies to new objects; Batch Replication handles existing/failed ones.

</details>

**Q29.** Access patterns are unknown and change often. Minimize cost without retrieval fees.
A) Standard-IA B) Intelligent-Tiering C) Glacier Instant D) One Zone-IA

<details><summary>Answer</summary>

**B** — Auto-tiering with no retrieval charges.

</details>

---

## 11. S3 Advanced

- **Lifecycle**: transition (→ IA after 60 d, → Glacier after 180 d) + expiration (old versions, **incomplete multi-part uploads**, logs); filter by prefix or tags
- **Storage Class Analysis**: recommends Standard → Standard-IA only
- **Requester Pays**: requester pays transfer (must be authenticated)
- **Event notifications** → SNS, SQS, Lambda (need resource policies) or **EventBridge** (advanced filters, 18+ targets, archive/replay)
- Performance: **3,500 PUT / 5,500 GET per second per prefix**; **multi-part** (>100 MB recommended); **Transfer Acceleration** (edge → AWS backbone); **byte-range fetches**
- **Batch Operations**: bulk copy/encrypt/tag/restore/Lambda (list via **S3 Inventory** + Athena)
- **Storage Lens**: org-wide analytics (free 14 days / advanced 15 months)

**Q30.** Users worldwide upload large files to one bucket in us-east-1 slowly.
A) CRR B) Transfer Acceleration + multi-part upload C) CloudFront signed URLs D) Requester Pays

<details><summary>Answer</summary>

**B** — Upload to nearest edge, then AWS backbone; multi-part parallelizes.

</details>

**Q31.** Storage costs grow from abandoned multi-part uploads.
A) Versioning B) Lifecycle rule to abort incomplete multi-part uploads C) Intelligent-Tiering D) Object Lock

<details><summary>Answer</summary>

**B** — Lifecycle expiration can delete incomplete multi-part uploads.

</details>

**Q32.** Encrypt 50 million existing unencrypted objects with least effort.
A) Script with SDK B) S3 Batch Operations C) Re-upload D) Default encryption

<details><summary>Answer</summary>

**B** — Batch Operations; default encryption only affects new objects.

</details>

---

## 12. S3 Security

| Encryption | Key | Remember |
|---|---|---|
| **SSE-S3** | AWS owned, AES-256 | **Default** |
| **SSE-KMS** | KMS | User control + **CloudTrail audit**; counts against **KMS API quota** |
| **DSSE-KMS** | KMS + S3 | Two layers for compliance |
| **SSE-C** | Customer key sent each request | **HTTPS only**, AWS never stores key |
| Client-side | Customer | Encrypt before upload |

- Force HTTPS → bucket policy `aws:SecureTransport`; force encryption → bucket policy (evaluated before default encryption)
- **CORS**: cross-origin browser requests need `Access-Control-Allow-Origin` on the *other* bucket
- **MFA Delete**: permanent version delete + suspend versioning; only **root/bucket owner** enables; needs versioning
- **Access logs** → another bucket, same region (never same bucket → loop)
- **Pre-signed URLs**: inherit creator's permissions; console max 12 h, CLI max 7 days
- **Object Lock** (versioning): **Compliance** (nobody, not even root, can delete) / **Governance** (special permission can) / **Legal Hold**. **Glacier Vault Lock** = locked WORM policy
- **Access Points** (per-team policies + DNS, VPC origin) · **Object Lambda** (redact PII, convert, resize on read)

**Q33.** Compliance requires auditing every use of the encryption key for S3 objects.
A) SSE-S3 B) SSE-KMS C) SSE-C D) Client-side

<details><summary>Answer</summary>

**B** — KMS key usage is logged in CloudTrail.

</details>

**Q34.** Financial records must not be deleted or overwritten by anyone, including root, for 7 years.
A) MFA Delete B) Object Lock Governance C) Object Lock Compliance mode D) Bucket policy deny

<details><summary>Answer</summary>

**C** — Compliance mode can't be bypassed or shortened, even by root.

</details>

**Q35.** Premium videos in a private bucket should be downloadable only by logged-in users for 1 hour.
A) Public bucket B) Pre-signed URLs C) CORS D) Access logs

<details><summary>Answer</summary>

**B** — Generate time-limited pre-signed URLs per user.

</details>

**Q36.** Website in bucket A loads fonts from bucket B and the browser blocks them.
A) Bucket policy on A B) CORS configuration on bucket B C) Versioning D) Transfer Acceleration

<details><summary>Answer</summary>

**B** — The cross-origin target (B) must allow A's origin via CORS.

</details>

---

## 13. CloudFront & Global Accelerator

- **CloudFront**: CDN, edge caching, DDoS protection with **Shield + WAF**. Origins: **S3 (secure with OAC)**, **VPC origin** (private ALB/NLB/EC2), custom HTTP (S3 website, public ALB)
- **Geo restriction** (allow/block countries) · **Cache invalidation** (`/*` or path) · CloudFront for static global content vs **S3 CRR** for dynamic content in few regions
- **Global Accelerator**: **2 Anycast static IPs**, AWS backbone, **TCP/UDP**, health checks → failover < 1 min. Good for gaming (UDP), IoT, VoIP, static-IP HTTP, fast regional failover. **No caching**

**Q37.** A multiplayer game uses UDP and needs fast global performance with fixed IPs.
A) CloudFront B) Global Accelerator C) Route 53 latency D) ALB

<details><summary>Answer</summary>

**B** — UDP + static IPs + AWS backbone = Global Accelerator.

</details>

**Q38.** S3 content served through CloudFront must not be accessible directly from S3.
A) Make bucket public B) Origin Access Control + bucket policy allowing only CloudFront C) Pre-signed URLs D) CORS

<details><summary>Answer</summary>

**B** — OAC restricts the bucket to the CloudFront distribution.

</details>

**Q39.** You updated `index.html` in S3 but users still see the old page.
A) Wait for versioning B) Create a CloudFront invalidation C) Enable CRR D) Restart CloudFront

<details><summary>Answer</summary>

**B** — Invalidate the path to bypass the TTL.

</details>

---

## 14. Storage Extras

- **Snowball Edge** (Storage Optimized 210 TB / Compute Optimized): use if network transfer > 1 week; edge computing (EC2/Lambda on device). **Snowball → S3 → lifecycle → Glacier** (can't import directly to Glacier)
- **FSx**: **Windows** (SMB, NTFS, **Active Directory**, Multi-AZ) · **Lustre** (HPC/ML, S3 integration; **scratch** vs **persistent**) · **NetApp ONTAP** (NFS/SMB/iSCSI, broad compatibility) · **OpenZFS** (NFS, 1M IOPS)
- **Storage Gateway**: **S3 File Gateway** (NFS/SMB → S3, local cache, AD) · **Volume Gateway** (iSCSI, cached/stored, EBS snapshots) · **Tape Gateway** (VTL → S3/Glacier)
- **Transfer Family**: FTP/FTPS/SFTP into **S3 or EFS**
- **DataSync**: scheduled sync on-prem (agent) → S3/EFS/FSx; AWS↔AWS (no agent); keeps permissions/metadata

**Q40.** Windows apps need a shared SMB file system integrated with Active Directory.
A) EFS B) FSx for Windows File Server C) FSx for Lustre D) S3

<details><summary>Answer</summary>

**B** — EFS is Linux-only NFS.

</details>

**Q41.** On-prem servers must keep using NFS while data is stored in S3, with frequently used files cached locally.
A) Volume Gateway B) S3 File Gateway C) DataSync D) Snowball

<details><summary>Answer</summary>

**B** — File Gateway exposes S3 over NFS/SMB with local cache.

</details>

**Q42.** 300 TB must move to AWS; the link is 100 Mbps and the deadline is 3 weeks.
A) Site-to-Site VPN B) Direct Connect C) Snowball Edge D) DataSync over internet

<details><summary>Answer</summary>

**C** — Over 100 Mbps it would take ~9 months; Direct Connect setup takes over a month.

</details>

**Q43.** An HPC workload processes data in S3 and needs a high-performance file system.
A) EFS Max I/O B) FSx for Lustre linked to S3 C) FSx Windows D) EBS gp3

<details><summary>Answer</summary>

**B** — Lustre is built for HPC and reads/writes S3 directly.

</details>

---

## 15. SQS, SNS, Kinesis, MQ

- **SQS Standard**: unlimited throughput, retention 4 d (max 14), 1,024 KB message, **at-least-once**, best-effort order. Consumers poll up to 10, then **DeleteMessage**
- **Visibility timeout** 30 s default (too low → duplicates; `ChangeMessageVisibility` to extend). **Long polling** 1–20 s reduces cost and empty responses
- **SQS FIFO**: order by **Message Group ID**, exactly-once via **Deduplication ID**, 300 msg/s (3,000 batched)
- ASG scales on `ApproximateNumberOfMessages`; SQS as **write buffer** for DB
- **SNS**: pub/sub, 12.5 M subs/topic; subscribers SQS, Lambda, Firehose, email, SMS, HTTP. **Fan-out SNS → many SQS** (queue policy must allow SNS; cross-region OK). **Filter policies**. SNS FIFO + SQS FIFO for ordered fan-out
- **Kinesis Data Streams**: real-time, retention **up to 365 d**, **replay**, ordering per shard/partition key; shard 1 MB/s in, 2 MB/s out; provisioned or on-demand
- **Data Firehose**: near real-time, serverless, loads to **S3, Redshift, OpenSearch**, Splunk, HTTP; Lambda transforms; **no storage/replay**
- **Amazon MQ**: managed ActiveMQ/RabbitMQ for **MQTT, AMQP, STOMP** apps migrating from on-prem; active/standby Multi-AZ

**Q44.** Some orders are processed twice because processing takes longer than expected.
A) Use long polling B) Increase visibility timeout C) Use SNS D) Decrease retention

<details><summary>Answer</summary>

**B** — Message becomes visible again before the consumer finishes.

</details>

**Q45.** One upload event must trigger fraud check, shipping and analytics independently.
A) One SQS queue B) SNS topic with 3 SQS subscriptions (fan-out) C) Kinesis D) Step Functions

<details><summary>Answer</summary>

**B** — Fan-out decouples consumers and keeps messages durable in each queue.

</details>

**Q46.** Clickstream data must be analyzed in real time and reprocessed later if needed.
A) SQS B) SNS C) Kinesis Data Streams D) Data Firehose

<details><summary>Answer</summary>

**C** — Real-time with replay. Firehose is near real-time and cannot replay.

</details>

**Q47.** Stream IoT data into S3 as Parquet with no code to manage.
A) Kinesis Data Streams + EC2 B) Data Firehose with format conversion C) SQS D) MQ

<details><summary>Answer</summary>

**B** — Firehose converts to Parquet/ORC and delivers to S3.

</details>

**Q48.** An on-prem app uses AMQP and must move to AWS without rewriting messaging code.
A) SQS B) SNS C) Amazon MQ D) Kinesis

<details><summary>Answer</summary>

**C** — MQ supports open protocols; SQS/SNS are proprietary APIs.

</details>

---

## 16. Containers

- **Docker** image → registry (**ECR** private/public, Docker Hub) → container. Shares host OS (lighter than VMs)
- **ECS launch types**: **EC2** (you manage instances, ECS agent) vs **Fargate** (serverless, just task definitions)
- IAM: **EC2 instance profile** (agent: ECR pull, CloudWatch logs) vs **ECS Task Role** (per-task app permissions)
- LB: ALB (most), NLB (high throughput / PrivateLink). Storage: **EFS** (EC2 + Fargate, multi-AZ shared)
- **Service Auto Scaling** (tasks: CPU, memory, ALB requests) ≠ EC2 ASG; **Capacity Providers** add EC2 capacity
- Patterns: EventBridge → run task (on S3 upload / schedule), SQS-driven service
- **EKS**: managed Kubernetes (cloud-agnostic) — managed node groups, self-managed nodes, Fargate; storage via **CSI** (EBS, EFS, FSx)

**Q49.** Run containers without managing servers or clusters of EC2.
A) ECS EC2 launch type B) ECS on Fargate C) EKS self-managed nodes D) Elastic Beanstalk single instance

<details><summary>Answer</summary>

**B** — Fargate is serverless compute for containers.

</details>

**Q50.** Two ECS services need different permissions (one S3, one DynamoDB).
A) One instance profile with both B) Separate ECS Task Roles C) Access keys in env vars D) Security groups

<details><summary>Answer</summary>

**B** — Task roles give least-privilege per task.

</details>

**Q51.** A company already runs Kubernetes on-prem and wants the same tooling on AWS.
A) ECS B) EKS C) Fargate alone D) Lambda

<details><summary>Answer</summary>

**B** — EKS is managed Kubernetes.

</details>

---

## 17. Serverless

- **Lambda**: up to **15 min**, 128 MB–**10 GB** RAM (more RAM = more CPU), `/tmp` up to 10 GB, **1,000 concurrent** per region (reserved concurrency = cap), zip 50 MB / unzipped 250 MB, container images allowed. Pay per request + GB-second
- Throttle: sync → **429**; async → retries up to 6 h → **DLQ**
- **Cold starts** → **Provisioned Concurrency**; **SnapStart** (Java, Python, .NET) up to 10x faster
- **Lambda in VPC** → ENI in your subnets (needed for private RDS/ElastiCache); use **RDS Proxy** for DB connections
- RDS for PostgreSQL / Aurora MySQL can **invoke Lambda**; **RDS Event Notifications** describe the instance, not data
- **CloudFront Functions** (JS, < 1 ms, viewer req/resp, headers, URL rewrite, JWT) vs **Lambda@Edge** (Node/Python, 5–10 s, origin req/resp too, network access)
- **DynamoDB**: serverless NoSQL, item **400 KB**, partition key (+ sort key). **Provisioned** (RCU/WCU, auto scaling) vs **On-demand** (spiky). **DAX** (µs cache, no code change). **Streams** (24 h) → Lambda. **Global Tables** (active-active, needs Streams). **TTL**. **PITR 35 days**. Export/import S3 without capacity use
- **API Gateway**: REST/WebSocket, throttling, API keys, caching, stages, transforms. Endpoint: **Edge-optimized** (default, cert in us-east-1) / **Regional** / **Private** (VPC endpoint). Auth: IAM, **Cognito**, custom authorizer
- **Step Functions**: visual workflows, retries, parallel, human approval
- **Cognito**: **User Pools** (sign-in, MFA, social/SAML; integrates API Gateway/ALB) vs **Identity Pools** (temporary **AWS credentials**, row-level security)

**Q52.** A job runs 40 minutes. Lambda?
A) Yes, increase timeout B) No — use ECS/Fargate or AWS Batch C) Use Lambda@Edge D) Use Step Functions with one Lambda

<details><summary>Answer</summary>

**B** — Lambda max is 15 minutes.

</details>

**Q53.** A Java Lambda API has slow first requests after idle.
A) Increase timeout B) Provisioned Concurrency or SnapStart C) Reserved concurrency D) Move to VPC

<details><summary>Answer</summary>

**B** — Both remove/minimize cold start init time. Reserved concurrency only caps.

</details>

**Q54.** Lambda must query an RDS DB in a private subnet.
A) Make RDS public B) Configure Lambda with VPC subnets + security group (optionally RDS Proxy) C) Use NAT Gateway only D) API Gateway private endpoint

<details><summary>Answer</summary>

**B** — By default Lambda runs outside your VPC.

</details>

**Q55.** DynamoDB reads need microsecond latency without changing application logic.
A) ElastiCache B) DAX C) Global Tables D) On-demand mode

<details><summary>Answer</summary>

**B** — DAX is API-compatible with DynamoDB.

</details>

**Q56.** Mobile app users must upload files directly to their own S3 prefix.
A) IAM user per customer B) Cognito Identity Pools with IAM policy variables C) Public bucket D) Access keys in the app

<details><summary>Answer</summary>

**B** — Identity Pools issue temporary AWS credentials scoped per user.

</details>

**Q57.** Add security headers to every CloudFront response at the lowest cost and latency.
A) Lambda@Edge B) CloudFront Functions C) API Gateway D) ALB rules

<details><summary>Answer</summary>

**B** — Lightweight header manipulation at viewer response = CloudFront Functions.

</details>

---

## 18. Serverless Architectures

- **Mobile app**: Cognito → API Gateway → Lambda → DynamoDB (+ **DAX**, + **API Gateway cache**); direct S3 access via **Identity Pools**
- **Serverless website**: **CloudFront + S3 (OAC)** for static, API Gateway + Lambda + DynamoDB **Global Tables** for dynamic; welcome email = **DynamoDB Streams → Lambda → SES**; thumbnails = S3 event → Lambda
- **Microservices**: sync (API Gateway, ELB) and async (SQS, SNS, Kinesis, S3 events) per service
- **Software update spikes on EC2**: put **CloudFront** in front — no app change, cache static files, ASG scales less

**Q58.** Send a welcome email whenever a new user row is added to DynamoDB.
A) Cron job scanning the table B) DynamoDB Streams → Lambda → SES C) SNS on table D) CloudTrail

<details><summary>Answer</summary>

**B** — Streams react to item-level changes in near real time.

</details>

**Q59.** An EC2 app distributing static software updates is overloaded on release days. Cheapest fix without changing the app?
A) Bigger instances B) CloudFront in front C) Move to Lambda D) Add Read Replicas

<details><summary>Answer</summary>

**B** — Cache static files at the edge.

</details>

---

## 19. Choosing Databases

| Need | Service |
|---|---|
| Relational, joins, OLTP | RDS / Aurora |
| Key-value, serverless, ms | DynamoDB (+ DAX) |
| In-memory cache | ElastiCache |
| **MongoDB** (JSON documents) | **DocumentDB** |
| **Graph** (social, fraud, recommendations) | **Neptune** (+ Streams) |
| **Apache Cassandra** (CQL) | **Keyspaces** |
| **Time series** (IoT) | **Timestream** |
| Data warehouse (OLAP) | Redshift |
| Full-text search | OpenSearch |
| Objects | S3 |

**Q60.** Store and query highly connected social network data (friends of friends).
A) DynamoDB B) Neptune C) Redshift D) RDS

<details><summary>Answer</summary>

**B** — Graph database.

</details>

**Q61.** Migrate an existing MongoDB workload to a managed AWS service.
A) DynamoDB B) DocumentDB C) Keyspaces D) Aurora

<details><summary>Answer</summary>

**B** — MongoDB-compatible.

</details>

**Q62.** Store trillions of IoT sensor readings per day and analyze trends over time.
A) RDS B) Timestream C) Neptune D) ElastiCache

<details><summary>Answer</summary>

**B** — Purpose-built time series DB.

</details>

---

## 20. Data & Analytics

- **Athena**: serverless SQL on S3 ($5/TB scanned). Save cost: **Parquet/ORC**, compression, **partitioning**, files > 128 MB. **Federated query** via Lambda connectors
- **Redshift**: OLAP, columnar, leader + compute nodes, provisioned or serverless; load via **COPY from S3** (batch large inserts); snapshots auto-copy cross-region; **Spectrum** queries S3 without loading
- **OpenSearch**: search any field / partial match; ingest from Firehose, CloudWatch Logs; DynamoDB → Streams → Lambda → OpenSearch
- **EMR**: Hadoop/Spark clusters; master, core, task nodes (task → Spot)
- **QuickSight**: serverless BI dashboards, SPICE, column-level security; users/groups are QuickSight-only
- **Glue**: serverless **ETL** (CSV → Parquet), **Data Catalog** + crawlers (used by Athena, Redshift Spectrum, EMR), job bookmarks, DataBrew, Studio, streaming ETL
- **Lake Formation**: data lake in days on Glue; **row/column-level access control**
- **Managed Service for Apache Flink**: stream processing from KDS/MSK (**not Firehose**)
- **MSK**: managed Kafka (partitions, configurable message size > 1 MB, EBS storage); MSK Serverless
- **Pipeline**: IoT Core → KDS → Firehose → S3 → (SQS/Lambda) → Athena → S3 reports → QuickSight / Redshift

**Q63.** Analysts want SQL queries on CSV logs in S3 with no servers.
A) Redshift B) Athena C) EMR D) RDS

<details><summary>Answer</summary>

**B** — Serverless SQL on S3.

</details>

**Q64.** Athena costs are high. Two best improvements? (Choose 2)
A) Convert to Parquet B) Partition data by date C) Use smaller files D) Use JSON E) Use Glacier

<details><summary>Answer</summary>

**A, B** — Columnar + partitions = less data scanned.

</details>

**Q65.** Redshift must join warehouse tables with years of history kept cheaply in S3, without loading it.
A) COPY B) Redshift Spectrum C) Athena federated D) Glue

<details><summary>Answer</summary>

**B** — Spectrum queries S3 data from Redshift.

</details>

**Q66.** A data lake needs column- and row-level permissions for different analyst teams.
A) S3 bucket policies B) Lake Formation C) QuickSight D) IAM groups only

<details><summary>Answer</summary>

**B** — Fine-grained, centralized data lake permissions.

</details>

---

## 21. Machine Learning (match input → service)

| Service | Does |
|---|---|
| Rekognition | Images/video: objects, faces, text, **content moderation** |
| Transcribe | Speech → text (PII redaction) |
| Polly | Text → speech (lexicons, SSML) |
| Translate | Language translation |
| Lex | Chatbots (ASR + NLU) |
| Connect | Cloud contact center |
| Comprehend | NLP, sentiment, entities (**Comprehend Medical**: PHI) |
| SageMaker AI | Build/train/deploy your own models |
| Kendra | ML document search (natural language) |
| Personalize | Real-time recommendations |
| Textract | Text, forms, tables from scanned documents |

**Q67.** Extract fields from scanned invoices and forms.
A) Rekognition B) Textract C) Comprehend D) Kendra

<details><summary>Answer</summary>

**B**

</details>

**Q68.** Detect inappropriate images uploaded by users, with human review for low-confidence results.
A) Rekognition content moderation + Augmented AI (A2I) B) Comprehend C) Textract D) SageMaker only

<details><summary>Answer</summary>

**A**

</details>

**Q69.** Analyze customer emails for positive or negative sentiment.
A) Transcribe B) Comprehend C) Polly D) Lex

<details><summary>Answer</summary>

**B**

</details>

---

## 22. Monitoring & Audit

- **CloudWatch Metrics**: namespaces, dimensions (30), **custom metrics** (RAM needs **Unified Agent**), Metric Streams → Firehose
- **CloudWatch Logs**: groups/streams, retention, encryption; **Logs Insights** (query, not real-time); **S3 export** (up to 12 h); **Subscription filters** (real-time → KDS, Firehose, Lambda; cross-account aggregation). EC2 needs an agent
- **Alarms**: OK / INSUFFICIENT_DATA / ALARM; targets EC2 actions (**recover** on `StatusCheckFailed_System`), ASG, SNS; **composite alarms** reduce noise
- **EventBridge**: schedules, event patterns, custom/partner buses, archive/replay, schema registry, cross-account via resource policy
- **Insights**: Container, Lambda (layer), **Contributor** (top-N), Application
- **CloudTrail**: API history (console/SDK/CLI/services), on by default, **90 days** (→ S3 + Athena for longer). Management events (default), **data events** (S3 object, Lambda invoke — off by default), **Insights** (unusual activity). CloudTrail + EventBridge = alert on any API call
- **Config**: per-region config history + compliance rules (managed or Lambda custom), **doesn't block**, remediation via **SSM Automation**
- CloudWatch = performance · CloudTrail = **who did what** · Config = **what changed / compliant?**

**Q70.** A production S3 bucket was deleted. How do you find who did it?
A) CloudWatch Metrics B) CloudTrail C) Config D) Trusted Advisor

<details><summary>Answer</summary>

**B** — CloudTrail records the DeleteBucket API call and the identity.

</details>

**Q71.** Detect any security group that allows SSH from 0.0.0.0/0 and auto-remediate.
A) CloudTrail B) AWS Config rule + SSM Automation remediation C) GuardDuty D) Inspector

<details><summary>Answer</summary>

**B** — Config evaluates compliance and can trigger remediation.

</details>

**Q72.** Monitor memory utilization of EC2 instances.
A) Default EC2 metrics B) CloudWatch Unified Agent custom metric C) CloudTrail D) VPC Flow Logs

<details><summary>Answer</summary>

**B** — RAM is not a default EC2 metric.

</details>

**Q73.** Logs from 20 accounts must stream in near real time to one central S3 bucket.
A) S3 export tasks B) Subscription filters → Kinesis Data Streams/Firehose in central account C) Logs Insights D) CloudTrail

<details><summary>Answer</summary>

**B** — Cross-account subscription with a destination in the central account.

</details>

---

## 23. Advanced Identity

- **Organizations**: management + member accounts, OUs, **consolidated billing**, shared RI/Savings Plan discounts, account creation API
- **SCPs**: restrict accounts/OUs (users **and root of member accounts**); **don't apply to management account**; need explicit allow down the path; Deny wins. **Tag Policies** standardize tags
- Conditions: `aws:SourceIp`, `aws:RequestedRegion`, `ec2:ResourceTag`, `aws:MultiFactorAuthPresent`, **`aws:PrincipalOrgID`** (resource policy: only my org)
- S3 ARNs: `s3:ListBucket` → `arn:aws:s3:::bucket`; object actions → `arn:aws:s3:::bucket/*`
- **Assume role** = give up your permissions; **resource-based policy** = keep them (S3, SNS, SQS, Lambda)
- **Permission boundaries** (users/roles): max permissions; prevent privilege escalation for one user
- **IAM Identity Center**: SSO to all org accounts + SAML apps; **permission sets**; ABAC; identity source built-in, AD, Okta
- **Directory Services**: **Managed Microsoft AD** (real AD in AWS, trust with on-prem) · **AD Connector** (proxy to on-prem) · **Simple AD** (standalone, no on-prem join)
- **Control Tower**: multi-account landing zone; guardrails **preventive (SCP)** + **detective (Config)**

**Q74.** Prevent every account in the Dev OU from launching resources outside eu-west-1.
A) IAM policy per user B) SCP on the Dev OU with `aws:RequestedRegion` deny C) Config rule D) Permission boundary

<details><summary>Answer</summary>

**B** — SCPs restrict all identities in the accounts, including their root users.

</details>

**Q75.** An S3 bucket should be accessible only by principals from accounts in your AWS Organization.
A) List all account IDs B) Bucket policy condition `aws:PrincipalOrgID` C) SCP D) VPC endpoint

<details><summary>Answer</summary>

**B**

</details>

**Q76.** Developers may create IAM roles for their apps but must never grant more than a defined set of permissions.
A) SCP B) Permission boundary C) Access Advisor D) MFA

<details><summary>Answer</summary>

**B** — Boundaries cap the max permissions and stop privilege escalation.

</details>

**Q77.** Employees should log in once with corporate AD credentials to many AWS accounts.
A) IAM users in each account B) IAM Identity Center with AD (AD Connector / Managed AD) C) Cognito User Pools D) Root accounts

<details><summary>Answer</summary>

**B**

</details>

---

## 24. Security & Encryption

- Encryption **in flight** (TLS), **server-side at rest**, **client-side**
- **KMS**: symmetric (AES-256, AWS services) / asymmetric (sign/verify, outside AWS). Keys: AWS owned (free), AWS managed (free, rotate yearly), customer managed ($1/mo, optional rotation), imported (manual rotation). **Key policies** required for access; audit via CloudTrail. Keys are **regional** → snapshot cross-region = **re-encrypt**. Cross-account snapshot = share snapshot + **key policy** for the CMK. **Multi-Region keys** (same key ID) for global client-side encryption
- **SSM Parameter Store**: config + secrets, hierarchy, versioning, free standard (4 KB) / advanced (8 KB, TTL policies)
- **Secrets Manager**: **automatic rotation** (Lambda), RDS integration, multi-region replicas
- **ACM**: free public TLS certs, auto-renew; on ELB, CloudFront, API Gateway (**not EC2**). DNS validation preferred. Imported certs don't auto-renew. **CloudFront/Edge API → cert in us-east-1**
- **CloudHSM**: dedicated hardware, **you manage keys**, **FIPS 140-2 Level 3**, single tenant, Multi-AZ cluster; KMS custom key store
- **WAF** (L7): ALB, API Gateway, CloudFront, AppSync, Cognito — **not NLB**. IP sets, SQLi, XSS, geo, **rate-based rules**. Fixed IP + WAF → **Global Accelerator → ALB + WAF**
- **Shield** Standard (free, L3/L4) / Advanced ($3,000/mo, DDoS team, cost protection, auto L7 mitigation)
- **Firewall Manager**: org-wide WAF/Shield/SG/Network Firewall policies, auto-applies to new resources
- **GuardDuty**: ML threat detection from **CloudTrail, VPC Flow Logs, DNS logs** (+ optional EKS, RDS, S3, Lambda); crypto-mining finding
- **Inspector**: vulnerability scans for **EC2 (SSM agent), ECR images, Lambda**
- **Macie**: finds **PII in S3**

**Q78.** Database passwords must rotate automatically every 30 days.
A) Parameter Store standard B) Secrets Manager C) KMS D) IAM

<details><summary>Answer</summary>

**B** — Built-in rotation with Lambda and RDS integration.

</details>

**Q79.** Regulations require single-tenant hardware and full customer control of keys (FIPS 140-2 Level 3).
A) KMS customer managed key B) CloudHSM C) SSE-S3 D) Secrets Manager

<details><summary>Answer</summary>

**B**

</details>

**Q80.** Block SQL injection and limit each IP to 1,000 requests per 5 minutes on an ALB.
A) Security groups B) NACL C) AWS WAF with managed rules + rate-based rule D) Shield Standard

<details><summary>Answer</summary>

**C**

</details>

**Q81.** Detect compromised EC2 instances communicating with known malicious IPs, with no agents.
A) Inspector B) GuardDuty C) Macie D) Config

<details><summary>Answer</summary>

**B** — Analyzes VPC Flow Logs, DNS logs and CloudTrail.

</details>

**Q82.** Share an encrypted EBS snapshot with another account.
A) Share snapshot only B) Use AWS managed key `aws/ebs` C) Encrypt with a customer managed key, add the other account to the key policy, share the snapshot D) Make it public

<details><summary>Answer</summary>

**C** — AWS managed keys can't be shared cross-account.

</details>

**Q83.** Find credit card numbers stored in S3 buckets across the account.
A) GuardDuty B) Macie C) Inspector D) Athena

<details><summary>Answer</summary>

**B**

</details>

---

## 25. VPC & Networking

- **CIDR**: /32 = 1 IP, /24 = 256, /16 = 65,536. Private ranges: 10.0.0.0/8, **172.16.0.0/12** (default VPC), 192.168.0.0/16
- VPC: max 5 per region, CIDR **/16 to /28**, no overlap with on-prem. Subnet = 1 AZ, **5 reserved IPs** (need 29 hosts → /26)
- **IGW** (1 per VPC) + **route table** `0.0.0.0/0 → igw` = public subnet. **Bastion host** in public subnet for SSH
- **NAT Gateway**: managed, in public subnet, needs IGW, per AZ for HA, 5→100 Gbps, no SG. NAT Instance: disable source/dest check, you manage. **Regional NAT Gateway** spans AZs
- **NACL** (subnet, stateless, allow+deny, numbered rules first match, **ephemeral ports 1024–65535**) vs **SG** (instance, stateful, allow only)
- **VPC Peering**: non-transitive, no overlapping CIDR, update route tables, cross-account/region
- **VPC Endpoints**: **Gateway** (S3, DynamoDB, free, route table) · **Interface** (ENI + SG, most services, paid; needed from on-prem/other region)
- **Flow Logs**: VPC/subnet/ENI → S3/CloudWatch/Firehose. Inbound ACCEPT + outbound REJECT → **NACL**
- **Site-to-Site VPN**: VGW (AWS) + CGW (customer, public IP), enable **route propagation**, ICMP for ping. **VPN CloudHub**: hub-and-spoke multiple sites
- **Direct Connect**: dedicated private line (1–400 Gbps dedicated / 50 Mbps–25 Gbps hosted, > 1 month lead time), **not encrypted** (add VPN for IPsec), **DX Gateway** for multi-region VPCs, backup with VPN or second DX
- **Transit Gateway**: transitive hub for thousands of VPCs + VPN + DX, RAM sharing, route tables, **IP multicast**, **ECMP** to multiply VPN bandwidth
- **PrivateLink**: expose your service to other VPCs (NLB + ENI). **Traffic Mirroring**: copy ENI traffic to appliances
- **IPv6**: all public; IPv4 can't be disabled; out of IPs → add IPv4 CIDR. **Egress-only IGW** = outbound-only IPv6
- Costs: same AZ private IP free; cross-AZ/public IP/inter-region cost; minimize egress; **gateway endpoint cheaper than NAT** for S3
- **Network Firewall**: L3–L7 for the whole VPC (domain lists, IPS)

**Q84.** A subnet must hold 29 EC2 instances. Smallest CIDR?
A) /28 B) /27 C) /26 D) /25

<details><summary>Answer</summary>

**C** — /27 = 32 − 5 reserved = 27 usable (too few); /26 = 59 usable.

</details>

**Q85.** Private-subnet instances need to download OS patches from the internet with no inbound access.
A) IGW on private subnet B) NAT Gateway in a public subnet + route C) VPC endpoint D) Elastic IP on each instance

<details><summary>Answer</summary>

**B**

</details>

**Q86.** Private EC2 instances access S3 heavily; NAT Gateway data charges are high.
A) Interface endpoint B) Gateway VPC endpoint for S3 C) Bigger NAT D) Direct Connect

<details><summary>Answer</summary>

**B** — Free and keeps traffic private.

</details>

**Q87.** Block one malicious IP from reaching all instances in a subnet.
A) Security group deny rule B) NACL deny rule C) Route table D) IAM

<details><summary>Answer</summary>

**B** — Security groups cannot deny.

</details>

**Q88.** VPC A peers with B, and B peers with C. Can A reach C?
A) Yes B) No, peering is not transitive C) Only via IGW D) Only same region

<details><summary>Answer</summary>

**B** — Create A–C peering or use Transit Gateway.

</details>

**Q89.** Connect 50 VPCs and 3 on-prem sites with simple routing.
A) Full mesh peering B) Transit Gateway C) VPN CloudHub D) NAT Gateway

<details><summary>Answer</summary>

**B**

</details>

**Q90.** Direct Connect traffic must be encrypted.
A) It already is B) Run a Site-to-Site VPN over Direct Connect (IPsec) C) Use TLS only on LB D) Use DX Gateway

<details><summary>Answer</summary>

**B**

</details>

**Q91.** IPv6 instances must reach the internet but not be reachable from it.
A) NAT Gateway B) Egress-only Internet Gateway C) IGW D) VPC endpoint

<details><summary>Answer</summary>

**B**

</details>

---

## 26. Disaster Recovery & Migrations

- **RPO** = data loss window · **RTO** = downtime
- Strategies (cost ↑, RTO ↓): **Backup & Restore** → **Pilot Light** (core DB running) → **Warm Standby** (full stack at minimum) → **Multi-Site / Hot** (full production, active-active)
- **Elastic Disaster Recovery (DRS)**: continuous block-level replication of servers to AWS, failover/failback
- **DMS**: migrate with source online, **CDC** continuous replication, runs on replication instance, Multi-AZ option. **SCT** converts schema for **different engines** (not needed same engine)
- RDS MySQL → Aurora: snapshot restore or Aurora read replica + promote. External MySQL → **Percona XtraBackup** in S3. External PostgreSQL → S3 + `aws_s3` extension
- **Application Discovery Service** (agentless / agent) → **Migration Hub**; **MGN** = lift-and-shift rehost; **VM Import/Export**; **VMware Cloud on AWS**
- **AWS Backup**: central policies (plans), cross-region/account, PITR, tags; **Vault Lock** = WORM (even root can't delete)
- 200 TB on 100 Mbps ≈ 185 days; DX 1 Gbps ≈ 18.5 days; **Snowball ≈ 1 week**

**Q92.** RPO of hours is acceptable and budget is minimal.
A) Multi-Site B) Warm Standby C) Pilot Light D) Backup & Restore

<details><summary>Answer</summary>

**D**

</details>

**Q93.** Only the database is replicated continuously to AWS; app servers start when disaster hits.
A) Backup & Restore B) Pilot Light C) Warm Standby D) Multi-Site

<details><summary>Answer</summary>

**B**

</details>

**Q94.** Migrate on-prem Oracle to Aurora PostgreSQL with minimal downtime.
A) DMS only B) SCT for schema + DMS with CDC C) Snowball D) mysqldump

<details><summary>Answer</summary>

**B** — Different engine needs schema conversion.

</details>

**Q95.** Backups across 15 accounts must be centrally managed and immutable.
A) Lambda scripts B) AWS Backup with Organizations policies + Vault Lock C) EBS snapshots manually D) S3 versioning

<details><summary>Answer</summary>

**B**

</details>

---

## 27. More Architectures & Other Services

- **Caching layers**: CloudFront → API Gateway → ElastiCache/DAX → DB
- **Block an IP**: NACL (direct), WAF on ALB, **WAF on CloudFront** (behind CloudFront, NACL sees CloudFront IPs; geo restriction is country-level)
- **HPC**: Cluster placement group, **Enhanced Networking (ENA, up to 100 Gbps)**, **EFA** (Linux, MPI, OS bypass), FSx for Lustre, instance store, Spot, **AWS Batch**, **ParallelCluster**
- **HA single EC2**: ASG min/max/desired = 1 across 2 AZ + user data attaches **Elastic IP**; EBS via lifecycle hooks + snapshots
- **CloudFormation**: IaC, stacks, **service role** + `iam:PassRole` for least privilege
- **SES** (email) · **Pinpoint** (marketing campaigns, SMS, segments)
- **SSM**: **Session Manager** (no SSH/port 22/bastion), Run Command, Patch Manager, Maintenance Windows, Automation runbooks
- **Cost Explorer** (analyze, Savings Plan recommendations, 18-month forecast) · **Cost Anomaly Detection** (ML, root cause, SNS)
- **Outposts**: AWS racks on-prem (latency, data residency) · **AWS Batch**: Docker batch jobs, no time limit · **AppFlow**: SaaS (Salesforce, SAP, Slack) ↔ S3/Redshift · **Amplify**: full-stack web/mobile · **Instance Scheduler**: stop/start EC2/RDS on schedule

**Q96.** Admins need shell access to private EC2 instances without opening port 22 or a bastion.
A) EC2 Instance Connect B) SSM Session Manager C) VPN D) Public IPs

<details><summary>Answer</summary>

**B**

</details>

**Q97.** Tightly coupled HPC simulation using MPI needs lowest inter-node latency on Linux.
A) ENA only B) Elastic Fabric Adapter + cluster placement group C) Spread placement D) NLB

<details><summary>Answer</summary>

**B**

</details>

**Q98.** Behind CloudFront + ALB, block a single abusive IP.
A) NACL on ALB subnet B) Security group C) WAF web ACL on CloudFront with IP set D) CloudFront geo restriction

<details><summary>Answer</summary>

**C** — The ALB/NACL only see CloudFront IPs; geo restriction is per country.

</details>

**Q99.** Be alerted automatically when daily spend suddenly spikes, without setting thresholds.
A) AWS Budgets B) Cost Anomaly Detection C) Trusted Advisor D) Cost Explorer forecast

<details><summary>Answer</summary>

**B**

</details>

**Q100.** A user should deploy a stack that creates S3 buckets without having S3 permissions themselves.
A) Give S3 full access B) CloudFormation service role + user has `iam:PassRole` C) SCP D) Root account

<details><summary>Answer</summary>

**B**

</details>

---

## 28. Well-Architected & Exam Strategy

- Principles: stop guessing capacity, test at production scale, automate, evolutionary architectures, data-driven, game days
- **6 pillars**: Operational Excellence · Security · Reliability · Performance Efficiency · Cost Optimization · **Sustainability** (synergy, not trade-offs)
- **Well-Architected Tool** (free review) · **Trusted Advisor** (cost, performance, security, fault tolerance, service limits, operational excellence; full checks need Business/Enterprise support)
- Tips: elimination, don't over-think, read FAQs and whitepapers, practice exams

**Q101.** Which pillar covers "recover from failures and meet demand"?
A) Performance Efficiency B) Reliability C) Operational Excellence D) Cost

<details><summary>Answer</summary>

**B**

</details>

**Q102.** Check the account for open security groups, idle resources and service limits — with no installation.
A) Inspector B) Trusted Advisor C) GuardDuty D) Config

<details><summary>Answer</summary>

**B**

</details>

---

## Mixed Scenario Questions (exam style)

**Q103.** A web app must survive an AZ failure with minimal management. Database is MySQL. Choose the design.
A) EC2 in one AZ + RDS single AZ B) ALB + ASG across 2 AZs + RDS Multi-AZ C) Two EC2 with Elastic IPs D) Lambda + RDS Read Replica

<details><summary>Answer</summary>

**B**

</details>

**Q104.** Image uploads spike unpredictably; processing takes 2–5 minutes per image; no images may be lost. (Choose the most cost-effective decoupled design)
A) API → EC2 synchronous processing B) S3 upload → S3 event → SQS → ASG of EC2 (scale on queue depth, Spot) C) SNS only D) Kinesis + Redshift

<details><summary>Answer</summary>

**B** — S3 + SQS buffers spikes durably; Spot workers scale on queue length. (Lambda would also fit at < 15 min.)

</details>

**Q105.** A global news site serves mostly static content plus a dynamic API with low latency worldwide, least ops.
A) EC2 in every region B) CloudFront (S3 static + API Gateway/Lambda origin) with DynamoDB Global Tables C) One big EC2 D) Route 53 simple routing to one region

<details><summary>Answer</summary>

**B**

</details>

**Q106.** Encrypt data at rest for a new RDS instance with keys you can audit and rotate.
A) Enable encryption at creation using a KMS customer managed key B) Enable encryption later C) SSE-S3 D) CloudHSM only

<details><summary>Answer</summary>

**A** — RDS encryption must be set at launch; existing DBs need snapshot → encrypted restore.

</details>

**Q107.** An app needs a fixed public IP, WAF protection and global users.
A) NLB + WAF B) Global Accelerator → ALB with WAF C) CloudFront only D) Elastic IP on ALB

<details><summary>Answer</summary>

**B** — WAF doesn't support NLB; ALB has no static IP; GA provides static Anycast IPs.

</details>

**Q108.** Steady 24/7 workload for 3 years on m5 instances; the team may later switch instance sizes in the same family.
A) On-Demand B) Spot C) EC2 Instance Savings Plan / Reserved Instances (3 yr) D) Dedicated Hosts

<details><summary>Answer</summary>

**C** — Long-term steady usage; Savings Plans are flexible across sizes within the family.

</details>

**Q109.** A company wants to stop developers from disabling CloudTrail in any member account.
A) IAM policy in each account B) SCP denying `cloudtrail:StopLogging` / `DeleteTrail` C) Config rule D) GuardDuty

<details><summary>Answer</summary>

**B** — Preventive control across all accounts.

</details>

**Q110.** Store session state for a stateless web tier with single-digit ms latency and automatic expiry, serverless.
A) RDS B) DynamoDB with TTL C) EBS D) S3

<details><summary>Answer</summary>

**B** — (ElastiCache Redis with TTL is also correct when "serverless" isn't required.)

</details>

---

## File Index (full notes)

| # | File |
|---|---|
| 01 | [Getting Started](01-getting-started.md) |
| 02 | [IAM](02-iam.md) |
| 03 | [EC2 Basics](03-ec2-basics.md) |
| 04 | [EC2 Associate](04-ec2-associate.md) |
| 05 | [EC2 Storage](05-ec2-storage.md) |
| 06 | [ELB + ASG](06-high-availability-scalability.md) |
| 07 | [RDS, Aurora, ElastiCache](07-rds-aurora-elasticache.md) |
| 08 | [Route 53](08-route53.md) |
| 09 | [Classic Architectures](09-classic-solutions-architecture.md) |
| 10 | [S3](10-s3.md) |
| 11 | [S3 Advanced](11-s3-advanced.md) |
| 12 | [S3 Security](12-s3-security.md) |
| 13 | [CloudFront & Global Accelerator](13-cloudfront-global-accelerator.md) |
| 14 | [Storage Extras](14-storage-extras.md) |
| 15 | [Messaging](15-integration-messaging.md) |
| 16 | [Containers](16-containers.md) |
| 17 | [Serverless](17-serverless.md) |
| 18 | [Serverless Architectures](18-serverless-architectures.md) |
| 19 | [Databases](19-databases-in-aws.md) |
| 20 | [Data & Analytics](20-data-analytics.md) |
| 21 | [Machine Learning](21-machine-learning.md) |
| 22 | [Monitoring & Audit](22-monitoring-audit-cloudwatch-cloudtrail-config.md) |
| 23 | [Advanced Identity](23-advanced-identity.md) |
| 24 | [Security & Encryption](24-security-encryption.md) |
| 25 | [VPC](25-vpc.md) |
| 26 | [DR & Migrations](26-disaster-recovery-migrations.md) |
| 27 | [More Architectures & Other Services](27-more-solutions-architecture.md) |
| 28 | [Well-Architected & Exam Tips](28-well-architected-exam-tips.md) |
| 29 | [Course Outline Map + Extras](29-course-outline-map.md) |
