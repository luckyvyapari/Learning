# 27 — More Solutions Architecture & Other Services

## Patterns recap

### Lambda + SNS/SQS
- SNS → Lambda (async, retries, **DLQ**); SQS → Lambda (poll, retries, **DLQ on the queue**); **SQS FIFO + Lambda** is blocking per message group (ordering)
- **Fan-out**: instead of an app PUTting to multiple SQS queues (Option 1), PUT once to **SNS** with queues subscribed (Option 2)
- **S3 events** → SNS / SQS / Lambda (filter `*.jpg`; delivery in seconds, sometimes a minute+) or **EventBridge** (JSON filtering, 18+ destinations, archive/replay)
- **Intercept API calls**: CloudTrail → EventBridge → SNS (e.g. DynamoDB `DeleteTable`)
- **API Gateway → Kinesis Data Streams → Firehose → S3**

### Caching strategies (client → edge → app → DB)
**CloudFront** (edge, static), **API Gateway** cache, app logic cache (**Redis/Memcached**), **DAX** (DynamoDB). Trade-off: caching/TTL vs network, computation, cost, latency. Caching closer to the user = lower latency and cost; caching deeper = less stale data risk.

### Blocking an IP address

| Setup | Where to block |
|---|---|
| EC2 in public subnet | **NACL** deny rule (SGs only allow); optional firewall software on EC2 |
| **ALB** | ALB terminates connection → EC2 sees ALB's private IP. NACL (at ALB subnet), **ALB SG** can't deny; EC2 SG only allows ALB; use **WAF** or NACL on public subnet |
| **NLB** | NLB passes client IP through → EC2 SG / NACL (private subnet) can match the client IP; no connection termination |
| **ALB + WAF** | **WAF IP filtering** on the ALB |
| **ALB + CloudFront + WAF** | Clients appear as **CloudFront public IPs**: **geo restriction is NOT helpful** for a specific IP; use **WAF** on CloudFront for IP filtering; NACL/SG must allow CloudFront IPs |

## High Performance Computing (HPC)

Cloud is ideal: spin up huge resources fast, speed up time-to-result, pay only for use. Genomics, computational chemistry, financial risk, weather, ML/DL, autonomous driving.

| Area | Services |
|---|---|
| **Data transfer** | **Direct Connect** (GB/s private), **Snowball/Snowmobile** (PB), **DataSync** (on-prem ↔ S3/EFS/FSx Windows) |
| **Compute** | EC2 **CPU/GPU optimized**; **Spot Instances/Fleets** + Auto Scaling; **Cluster placement group** (same rack/AZ, 10 Gbps) |
| **Enhanced Networking (SR-IOV)** | Higher bandwidth, higher PPS, lower latency. **ENA** up to **100 Gbps**; Intel 82599 VF up to 10 Gbps (legacy) |
| **EFA (Elastic Fabric Adapter)** | ENA improved for HPC, **Linux only**; tightly coupled inter-node comms; **MPI** standard; bypasses Linux OS kernel for low-latency reliable transport |
| **Storage** | EBS (**io2 Block Express 256,000 IOPS**), **Instance Store** (millions of IOPS), S3, **EFS** (IOPS scale with size or provisioned), **FSx for Lustre** (millions of IOPS, S3-backed) |
| **Automation** | **AWS Batch** (multi-node parallel jobs across EC2), **AWS ParallelCluster** (open-source HPC cluster tool, text config, creates VPC/subnets, **enable EFA**) |

## Highly Available EC2 Instance

| Approach | How |
|---|---|
| Standby + alarm | CloudWatch Event/Alarm → Lambda starts **standby EC2** and re-attaches **Elastic IP** |
| **ASG (min 1, max 1, desired 1) across ≥ 2 AZ** | **EC2 user data** attaches the Elastic IP (instance role allows API calls; found by **tag**) → replacement instance takes over |
| **ASG + EBS** | **ASG terminate lifecycle hook** → snapshot EBS (+ tags); **ASG launch lifecycle hook** → create + attach volume from snapshot in the new AZ (EBS is AZ-locked) |

## Other Services

### CloudFormation
**Declarative IaC**: describe resources in a template; CF creates them in the right order. Benefits: **infrastructure as code** (reviewed via code), **cost** (resources tagged by stack, estimate cost from template, auto delete/recreate dev at 5 PM/8 AM), **productivity** (destroy/re-create on the fly, auto diagrams, no ordering worries), reuse templates, supports nearly all resources (+ **custom resources**). **Infrastructure Composer** visualizes the stack.
**Service Role**: IAM role CloudFormation assumes to create/update/delete resources — enables **least privilege** (user needs `cloudformation:*` + **`iam:PassRole`**, not the resource permissions).

### SES (Simple Email Service)
Fully managed **email** sending, globally, at scale; inbound/outbound; reputation dashboard, anti-spam feedback; stats (deliveries, bounces, opens); **DKIM, SPF**; shared/dedicated/customer-owned IPs; Console, API or **SMTP**. Transactional, marketing, bulk.

### Pinpoint
Scalable **2-way marketing** communications: email, **SMS**, push, voice, in-app; **segmentation + personalization**; replies; billions of messages/day; events stream to SNS/Firehose/CloudWatch Logs. vs **SNS/SES**: there you manage audience/content/schedule yourself; Pinpoint has **templates, schedules, segments, full campaigns**.

### Systems Manager (SSM)
| Feature | What |
|---|---|
| **Session Manager** | Secure shell on EC2/on-prem via **SSM Agent** — **no SSH, no bastion, no port 22, no keys**; Linux/macOS/Windows; logs to S3/CloudWatch Logs; IAM permissions |
| **Run Command** | Execute a document/script across many instances (resource groups) — no SSH; output to console/S3/CloudWatch Logs; SNS status notifications; IAM + CloudTrail; invokable via EventBridge |
| **Patch Manager** | Automate OS/app/security patching (EC2 + on-prem; Linux/macOS/Windows); on demand or via **Maintenance Windows**; patch compliance report |
| **Maintenance Windows** | Schedule + duration + registered instances + registered tasks (patching, drivers, software) |
| **Automation** | Common maintenance/deployment tasks (restart instance, create AMI, EBS snapshot) via **Runbooks (SSM Documents)**; triggered by console/CLI/SDK, **EventBridge**, **Maintenance Windows**, or **AWS Config remediation** |

### Cost Explorer
Visualize/understand/manage cost + usage over time; custom reports; high level (all accounts) or **monthly/hourly/resource level**; choose an optimal **Savings Plan** (alternative to Reserved Instances); **forecast usage up to 18 months**.

### Cost Anomaly Detection
**ML** detects unusual spend (one-time spikes or continuous increases) — learns your historic pattern, **no thresholds needed**; monitor services, member accounts, cost allocation tags, cost categories; reports with **root-cause analysis**; alerts via **SNS** (individual or daily/weekly summary).

### Outposts
**Server racks** with the same AWS infrastructure/services/APIs/tools **on-premises** (AWS sets up/manages; **you secure the physical rack**). Hybrid cloud. Benefits: **low-latency** to on-prem systems, **local data processing**, **data residency**, easier migration. Works with EC2, EBS, S3, EKS, ECS, RDS, EMR.

### AWS Batch
Fully managed **batch** processing at any scale (100,000s of jobs). Dynamically launches **EC2 / Spot** and provisions right compute/memory. Jobs are **Docker images** run on ECS, EKS, Fargate. Example: S3 upload → trigger → Batch → process → insert to S3.

| | Lambda | Batch |
|---|---|---|
| Time limit | **Yes (15 min)** | **None** |
| Runtimes | Limited | **Any (Docker image)** |
| Disk | Limited temp space | **EBS / instance store** |
| Infra | **Serverless** | Relies on **EC2** (can be AWS-managed) |

### AppFlow
Fully managed **SaaS ↔ AWS** data transfer. Sources: **Salesforce, SAP, Zendesk, Slack, ServiceNow**. Destinations: S3, Redshift, Snowflake, Salesforce. Schedule / event / on demand; filtering/validation; encrypted over internet or **PrivateLink**. No custom integration code.

### Amplify
Tools/services to build and deploy **full-stack web & mobile apps**: auth (**Cognito**), storage (S3), API (REST via API Gateway/GraphQL via **AppSync**), CI/CD, PubSub, analytics, AI/ML, monitoring; connect GitHub/CodeCommit/Bitbucket/GitLab; hosting via **CloudFront**.

### Instance Scheduler on AWS
**CloudFormation-deployed solution** (not a service) — auto start/stop **EC2, ASGs, RDS** to cut cost (up to **70%**, e.g. outside business hours). Schedules in **DynamoDB**, uses **tags + Lambda**, supports cross-account/cross-region.

## Exam Hints

- Block an IP behind CloudFront/ALB → **WAF IP filter** (NACL for direct)
- HPC networking → **Cluster placement group + EFA + FSx for Lustre**
- Keep one instance always alive with static IP → **ASG 1/1/1 + user-data EIP attach**
- SSH without opening port 22 → **SSM Session Manager**
- Cost spike alert → **Cost Anomaly Detection**; forecast → **Cost Explorer**
- On-prem AWS APIs/services locally → **Outposts**
- Long-running batch with Docker → **Batch** (not Lambda)
