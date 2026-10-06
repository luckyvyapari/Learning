# 22 — Monitoring, Audit & Performance (CloudWatch, EventBridge, CloudTrail, Config)

## CloudWatch Metrics

Metrics for every AWS service. **Metric** = variable (CPUUtilization, NetworkIn) belonging to a **namespace**; **dimension** = attribute (instance id, env) — **up to 30 per metric**; timestamps; dashboards; **custom metrics** (e.g. RAM).

**Metric Streams**: near-real-time stream to **Kinesis Data Firehose** → S3, Redshift, OpenSearch, Athena, or 3rd party (Datadog, Dynatrace, New Relic, Splunk, Sumo Logic); can filter a subset.

## CloudWatch Logs

- **Log group** (usually an application) → **log streams** (instances/files/containers)
- Expiration policy: never, or 1 day – 10 years
- Encrypted by default; KMS with own keys optional
- **Sources**: SDK, Logs Agent, **Unified Agent**, Elastic Beanstalk, ECS, Lambda, **VPC Flow Logs**, API Gateway, CloudTrail (filter), Route 53 DNS queries
- **Send to**: S3 (export), Kinesis Data Streams, Firehose, Lambda, OpenSearch

### Logs Insights
Query/analyze logs with a purpose-built query language; auto-discovers fields (AWS services, JSON); filter, aggregate, sort, limit; save queries → dashboards; **multiple log groups across accounts**. **A query engine, not real-time.**

### S3 Export
`CreateExportTask`; can take **up to 12 hours** to be available. **Not near real-time** → use subscriptions.

### Subscriptions (real-time)
**Subscription filter** → **Kinesis Data Streams, Firehose, Lambda** (→ OpenSearch, S3…). **Aggregate multi-account, multi-region** logs into one KDS/Firehose → S3. **Cross-account subscription**: sender subscription filter → recipient **destination** (access policy) with IAM role allowing `PutRecord`.

### EC2 + agents
**By default no EC2 logs go to CloudWatch** → run an agent (also works on-prem) with correct IAM permissions.

| Agent | Capability |
|---|---|
| **Logs Agent** (old) | Logs only → CloudWatch Logs |
| **Unified Agent** | Logs **plus system metrics**: **RAM**, processes, disk, swap, netstat, detailed CPU; centralized config via **SSM Parameter Store** |

Out-of-the-box EC2 metrics: CPU, disk, network (high level) — **not RAM**.

## CloudWatch Alarms

- States: **OK**, **INSUFFICIENT_DATA**, **ALARM**
- **Period** = evaluation window; high-resolution custom metrics: 10 s, 30 s, or multiples of 60 s
- **Targets**: **EC2 actions** (stop, terminate, reboot, **recover**), **Auto Scaling action**, **SNS**
- **Composite alarms**: monitor states of **multiple alarms** with AND/OR → reduce alarm noise
- Can alarm on **Logs Metric Filters**
- Test: `aws cloudwatch set-alarm-state --alarm-name "myalarm" --state-value ALARM --state-reason "testing purposes"`

### EC2 Instance Recovery
Status checks: **Instance** (VM), **System** (underlying hardware), **Attached EBS**. Alarm on `StatusCheckFailed_System` → **recover** instance. Recovery keeps **same private, public, Elastic IP, metadata, placement group**.

### Network Synthetic Monitor
Detect network issues (packet loss, latency, jitter) between AWS and **on-prem** (Direct Connect / VPN). **No agent**, ICMP/TCP probes, publishes to CloudWatch Metrics.

## CloudWatch Insights family

| Insight | Use |
|---|---|
| **Container Insights** | Metrics + logs for **ECS, EKS, Kubernetes on EC2, Fargate** (containerized CloudWatch agent for K8s) |
| **Lambda Insights** | System-level metrics (CPU, memory, disk, network) + **cold starts**, worker shutdowns; delivered as a **Lambda Layer** |
| **Contributor Insights** | **Top-N contributors** from logs (top talkers, bad hosts, URLs with most errors); works on VPC/DNS logs |
| **Application Insights** | **Automated dashboards** for app problems (EC2 with Java/.NET/IIS/DBs + EBS, RDS, ELB, ASG, Lambda, SQS, DynamoDB, S3, ECS, EKS, SNS, API Gateway); powered by SageMaker; findings → EventBridge + SSM OpsCenter |

## EventBridge (formerly CloudWatch Events)

- **Schedule** (cron) → e.g. Lambda hourly
- **Event pattern** → react to service events (e.g. IAM **root user sign-in** → SNS email)
- Targets: Lambda, SQS, SNS, Kinesis, Step Functions, ECS task, Batch, CodeBuild/CodePipeline, SSM, EC2 actions…
- Sources: EC2 state change, CodeBuild, S3 event, Trusted Advisor finding, **CloudTrail (any API call)**, schedule
- **Event buses**: Default (AWS services), **Partner** (SaaS), **Custom**; **resource-based policies** let other accounts/regions send (aggregate org events into one account)
- **Archive** events (indefinitely or set period) and **replay**
- **Schema Registry**: infers schemas from events, generate code, versioned

**Intercept API calls**: CloudTrail logs API → EventBridge → SNS alert (e.g. `DeleteTable` on DynamoDB, `AssumeRole`, `AuthorizeSecurityGroupIngress`).

## CloudTrail

**Governance, compliance, audit.** **Enabled by default.** History of API calls/events from **Console, SDK, CLI, AWS services**. Send to **CloudWatch Logs** or **S3**. Trail applies to **all regions (default)** or one. **Resource deleted? Check CloudTrail first.**

| Event type | Notes |
|---|---|
| **Management events** | Operations on resources (IAM `AttachRolePolicy`, EC2 `CreateSubnet`, `CreateTrail`). **Logged by default**; separate Read vs Write |
| **Data events** | **Not logged by default** (high volume): **S3 object-level** (GetObject, PutObject, DeleteObject), **Lambda Invoke** |
| **Insights events** | Detect **unusual activity**: inaccurate provisioning, hitting service limits, IAM action bursts, gaps in periodic maintenance. Baselines management events, analyzes **write** events; shown in console, sent to S3, EventBridge event |

**Retention**: **90 days**. For longer → log to **S3** and query with **Athena**.

## AWS Config

Audit and record **compliance and configuration changes** over time. Answers: *Is SSH open to the world? Any public buckets? How did my ALB config change?* **Per-region** service; aggregate across regions/accounts; store in S3 (query with Athena); SNS alerts.

- **Config Rules**: **75+ managed**, or **custom (Lambda)**. Evaluated on **each config change** and/or **periodically**. **Does NOT prevent actions (no deny)**
- Pricing: no free tier; **$0.003 per config item** recorded, **$0.001 per rule evaluation** (per region)
- Resource view: compliance, configuration, and **CloudTrail API calls** over time
- **Remediation**: **SSM Automation Documents** (managed or custom, can invoke Lambda), remediation retries
- **Notifications**: EventBridge on NON_COMPLIANT → Lambda/SNS/SQS; or SNS for all config/compliance changes

## CloudWatch vs CloudTrail vs Config

| | Purpose |
|---|---|
| **CloudWatch** | **Performance monitoring** (metrics, dashboards), events & alerting, log aggregation/analysis |
| **CloudTrail** | **Who did what** — API calls by everyone; trails; global |
| **Config** | **What changed / is it compliant** — config history + compliance rules |

**Example — ELB**: CloudWatch = incoming connections, error code %, performance dashboard · Config = SG rule changes, config changes, ensure SSL cert assigned · CloudTrail = who changed the LB via API.

## Exam Hints

- RAM metric on EC2 → **custom metric / Unified Agent**
- Who deleted/changed it → **CloudTrail**; is it compliant / config history → **Config**
- Real-time log processing → **Subscription filter**; analytics queries → **Logs Insights**
- Events older than 90 days → **S3 + Athena**
- Auto-recover failed EC2 hardware → alarm on `StatusCheckFailed_System` → **recover**
- React to any API call → **CloudTrail + EventBridge**
- Detect unusual API activity → **CloudTrail Insights**
