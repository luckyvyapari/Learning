# 17 — Serverless (Lambda, DynamoDB, API Gateway, Cognito, Step Functions)

**Serverless** = you don't manage/provision/see servers; you deploy code/functions. Started as FaaS (Lambda) and now includes anything managed: Lambda, DynamoDB, Cognito, API Gateway, S3, SNS & SQS, Kinesis Data Firehose, Aurora Serverless, Step Functions, Fargate.

## AWS Lambda

| EC2 | Lambda |
|---|---|
| Virtual servers, limited by RAM/CPU, always running, scaling = intervention | Virtual **functions**, limited by time, run **on demand**, **automatic scaling** |

- **Pricing**: pay per request + compute time. Free tier **1M requests + 400,000 GB-s/month**. After: **$0.20 per 1M requests**, **$1.00 per 600,000 GB-s** (billed per 1 ms)
- Up to **10 GB RAM** (more RAM → more CPU + network)
- Languages: Node.js, Python, Java, C#/PowerShell, Ruby, **Custom Runtime API** (Rust, Go); **Lambda Container Image** must implement Runtime API (ECS/Fargate preferred for arbitrary Docker images)
- Integrations: API Gateway, Kinesis, DynamoDB, S3, CloudFront, EventBridge/CloudWatch Events, CloudWatch Logs, SNS, SQS, Cognito
- Patterns: **thumbnail creation** (S3 → Lambda → thumbnail S3 + metadata in DynamoDB); **serverless cron** (EventBridge hourly → Lambda)

### Limits (per region)

| Limit | Value |
|---|---|
| Memory | 128 MB – **10 GB** |
| Max execution time | **900 s (15 min)** |
| Env variables | 4 KB |
| `/tmp` disk | 512 MB – 10 GB |
| Concurrent executions | **1,000** (can increase) |
| Deployment zip (compressed) | **50 MB** |
| Uncompressed code + deps | **250 MB** |

### Concurrency & Throttling
- Limit 1,000 concurrent executions; set **reserved concurrency** per function (also acts as a limit)
- Over limit → **Throttle**: **synchronous** → `429 ThrottleError`; **asynchronous** → auto retry (up to **6 hours**, exponential backoff 1 s → 5 min) then **DLQ**
- Without reservation, one noisy source can throttle others

### Cold Starts & Provisioned Concurrency
- **Cold start**: new instance loads code + runs init outside handler → first request slower
- **Provisioned Concurrency**: pre-allocated → no cold starts; manage with Application Auto Scaling
- **SnapStart**: up to **10x** faster startup at no extra cost for **Java, Python, .NET** — snapshot of initialized memory/disk state on version publish

### Lambda in a VPC
- **Default**: runs in an AWS-owned VPC → **cannot reach** RDS, ElastiCache, internal ELB in your VPC
- To access: configure **VPC ID + subnets + security groups** → Lambda creates an **ENI**
- **Lambda + RDS Proxy**: pools connections, prevents connection storms, reduces failover time 66%, IAM auth + Secrets Manager; Lambda **must be in the VPC** (RDS Proxy never public)
- **Invoke Lambda from RDS/Aurora**: supported for **RDS PostgreSQL and Aurora MySQL**; DB needs outbound network path (public/NAT/VPC endpoint) + permissions (Lambda resource policy + IAM)
- **RDS Event Notifications**: info about the DB *instance* (created/stopped/…), **not data**; near real-time (up to 5 min); to **SNS** or **EventBridge**

## Edge Functions (customization at the edge)

Code attached to CloudFront, runs close to users. Serverless, global, pay per use. Uses: security, SEO, A/B testing, auth, bot mitigation, real-time image transformation, routing across origins.

| | **CloudFront Functions** | **Lambda@Edge** |
|---|---|---|
| Runtime | JavaScript | Node.js, Python |
| Scale | **Millions** req/s | Thousands req/s |
| Triggers | Viewer request/response | Viewer **and Origin** request/response |
| Max time | **< 1 ms** | 5–10 s |
| Memory | 2 MB | 128 MB – 10 GB |
| Package | 10 KB | 1–50 MB |
| Network/file system/request body | **No** | **Yes** |
| Price | Free tier, ~1/6 of @Edge | No free tier |

- CloudFront Functions: cache key normalization, header manipulation, URL rewrites/redirects, JWT validation
- Lambda@Edge: longer execution, 3rd-party libs (SDK), external network, file system/body access. Author in **us-east-1**, CloudFront replicates

## DynamoDB

Fully managed **NoSQL**, multi-AZ replicated, transactions supported, single-digit ms, millions req/s, 100s TB, IAM-integrated, auto-scaling. **Standard** and **Infrequent Access (IA)** table classes.

- **Table** with **Primary Key** (decided at creation): **Partition Key** or **Partition Key + Sort Key**
- Items (rows) with attributes (can be added over time/null); **max item size 400 KB**
- Types: Scalar (String, Number, Binary, Boolean, Null), Document (List, Map), Set (String/Number/Binary Set)

| Capacity mode | How |
|---|---|
| **Provisioned** (default) | Set **RCU/WCU**; plan ahead; optional auto-scaling |
| **On-Demand** | Auto scales, no planning, pay per use, more expensive; **unpredictable/spiky workloads** |

### DAX (DynamoDB Accelerator)
Managed in-memory cache; **microseconds** latency; **no app code changes**; **5 min TTL** default. **DAX vs ElastiCache**: DAX = individual object cache + query/scan cache; ElastiCache = store aggregation results.

### Streams
Ordered item-level changes (create/update/delete). Uses: react in real time (welcome email), analytics, derivative tables, cross-region replication, Lambda trigger.

| DynamoDB Streams | Kinesis Data Streams |
|---|---|
| **24 h** retention | **1 year** |
| Limited consumers | Many consumers |
| Lambda triggers, KCL adapter | Lambda, Kinesis Data Analytics, Firehose, Glue Streaming ETL |

### Global Tables
**Active-active** multi-region replication; read/write in any region; **Streams must be enabled**.

### TTL
Auto-delete items after an **epoch timestamp** attribute. Reduce data, regulatory, session handling.

### Backups & S3 integration
- **PITR** (continuous, last **35 days**) → restore creates a **new table**
- **On-demand backups** (until deleted; no performance impact; manage in **AWS Backup**, cross-region copy) → new table
- **Export to S3** (requires PITR, any time in last 35 days, no read capacity used; DynamoDB JSON/ION) → analyze with Athena
- **Import from S3** (CSV/JSON/ION; no write capacity; creates new table; errors in CloudWatch Logs)

## API Gateway

Serverless API front door. REST + **WebSocket**, versioning, environments (dev/test/prod), auth, **API keys**, **throttling**, Swagger/OpenAPI import, request/response transform + validation, SDK generation, **response caching**.

**Integrations**: **Lambda** (REST API backed by Lambda) · **HTTP** (on-prem API, ALB — add rate limiting, caching, auth) · **AWS Service** (e.g. start Step Functions, post to SQS, put to Kinesis).

| Endpoint type | Use |
|---|---|
| **Edge-Optimized** (default) | Global clients, routed via CloudFront edges; API lives in one region |
| **Regional** | Same-region clients; can combine manually with CloudFront |
| **Private** | Only from your VPC via **interface VPC endpoint**; resource policy |

**Security**: IAM roles (internal), **Cognito** (external/mobile users), **Custom Authorizer**. Custom domain via **ACM**: Edge-optimized → cert in **us-east-1**; Regional → cert in the API's region. Route 53 CNAME or A-alias.

## Step Functions

Visual serverless **workflow orchestration**: sequence, parallel, conditions, timeouts, error handling, **human approval**. Integrates with Lambda, EC2, ECS, on-prem, API Gateway, SQS. Uses: order fulfillment, data processing.

## Cognito

| | User Pools (CUP) | Identity Pools (Federated Identities) |
|---|---|---|
| Purpose | **Sign-in** for app users (serverless user DB) | Give users **temporary AWS credentials** |
| Features | Username/email+password, password reset, email/phone verification, **MFA**, federated logins (Facebook, Google, SAML) | IAM policies defined in Cognito, customizable by `user_id` (**row-level security in DynamoDB**), default roles for authenticated/guest |
| Integrates with | **API Gateway, ALB** | User Pools or 3rd-party login as identity source; access S3/DynamoDB directly or via API Gateway |

Cognito vs IAM keywords: "**hundreds of users**", "**mobile users**", "**authenticate with SAML**".

## Exam Hints

- Lambda can't reach VPC resources by default → configure VPC (ENI)
- Too many DB connections from Lambda → **RDS Proxy**
- Slow first invocation → **Provisioned Concurrency / SnapStart**
- Run code at the edge, lightweight → **CloudFront Functions**; heavy/network → **Lambda@Edge**
- Cache DynamoDB reads, no code change → **DAX**
- Multi-region active-active NoSQL → **DynamoDB Global Tables**
- Mobile users directly access S3 → **Cognito Identity Pool**
- API caching/throttling/auth → **API Gateway**
