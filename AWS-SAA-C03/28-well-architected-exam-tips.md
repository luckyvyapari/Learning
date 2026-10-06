# 28 — Well-Architected, Trusted Advisor & Exam Tips

## Well-Architected Framework

### General guiding principles
- **Stop guessing capacity** needs
- **Test systems at production scale**
- **Automate** to make experimentation easy
- Allow **evolutionary architectures**
- Design based on **changing requirements**
- **Drive architectures using data**
- Improve through **game days** (simulate flash-sale days)

### 6 Pillars
| # | Pillar | Think |
|---|---|---|
| 1 | **Operational Excellence** | Run/monitor, improve procedures (CloudFormation, Config, CloudWatch) |
| 2 | **Security** | Protect data/systems (IAM, KMS, WAF, GuardDuty) |
| 3 | **Reliability** | Recover from failure, scale (Multi-AZ, ASG, DR) |
| 4 | **Performance Efficiency** | Right resources, scale (caching, serverless, right instance types) |
| 5 | **Cost Optimization** | Avoid unnecessary spend (Spot, Savings Plans, rightsizing) |
| 6 | **Sustainability** | Minimize environmental impact |

They're **not trade-offs to balance — they're a synergy**.

### Well-Architected Tool
**Free** tool to review workloads against the 6 pillars: select workload → answer questions → review against pillars → get advice (videos, docs), **generate report**, dashboard.

## Trusted Advisor

No install; **high-level account assessment** with recommendations in **6 categories**: **Cost optimization, Performance, Security, Fault tolerance, Service limits, Operational excellence**. **Business & Enterprise Support**: **full set of checks** + **programmatic access via AWS Support API**.

## More architectures
- Classic: EC2, ELB, RDS, ElastiCache. Serverless: S3, Lambda, DynamoDB, CloudFront, API Gateway
- See aws.amazon.com/architecture and aws.amazon.com/solutions

## Exam Tips

### How the exam works
| Item | Detail |
|---|---|
| Register | aws.training |
| Fee | **150 USD** |
| ID | One identity document |
| Rules | No notes, no pen, no speaking |
| Questions | **65 questions in 130 minutes** |
| Pass mark | **720 / 1000** |
| Tools | **Flag** questions for review; review all at end |
| Results | Pass/fail within **5 days** (usually less); overall score a few days later; **you don't see which answers were wrong** |
| Retake | After **14 days** |

### Strategy
- **Practice**: exam recommends **1+ year hands-on AWS** experience; review the material again if overwhelmed
- **Proceed by elimination**: most questions are **scenario based**; rule out known-wrong answers, pick the one that makes most sense; few trick questions; **don't over-think**
- **If a solution seems feasible but highly complicated, it's probably wrong**
- **Skim whitepapers**: Architecting for the Cloud — AWS Best Practices, Well-Architected Framework, AWS Disaster Recovery
- **Read each service's FAQ** (e.g. VPC FAQs) — covers many exam questions
- **Join the community**: course Q&A, practice tests, forums, blogs, local meetups, **re:Invent** videos

### Certification path (context)
Foundational (Cloud Practitioner) → **Associate** (Solutions Architect, Developer, SysOps, Data Engineer, ML Engineer, AI Practitioner…) → **Professional** (SA Pro, DevOps Pro; 2 years experience recommended) → **Specialty** (Security, Networking, ML…). Architecture track: **Solutions Architect** and Application Architect roles.

## Quick Decision Cheat Sheet (high-yield)

| Requirement | Answer |
|---|---|
| Decouple apps / buffer spikes | **SQS** |
| One event, many consumers | **SNS + SQS fan-out** |
| Real-time stream, replay | **Kinesis Data Streams** |
| Static IPs, TCP/UDP, global | **Global Accelerator** |
| Cache static content globally | **CloudFront** |
| Serverless SQL on S3 | **Athena** |
| Data warehouse | **Redshift** |
| Read scaling for DB | **Read Replicas** (Aurora up to 15) |
| DR for DB, automatic failover | **Multi-AZ** |
| Cross-region active-active NoSQL | **DynamoDB Global Tables** |
| Cross-region fast DB DR | **Aurora Global Database** |
| Shared Linux file system multi-AZ | **EFS** |
| Windows file share | **FSx for Windows** |
| HPC file system | **FSx for Lustre** |
| On-prem → S3 file access | **S3 File Gateway** |
| Move PB offline | **Snowball** |
| Private AWS-service access from VPC | **VPC Endpoint** (Gateway for S3/DynamoDB) |
| Many VPCs + on-prem | **Transit Gateway** |
| Dedicated line to AWS | **Direct Connect** |
| Layer 7 attacks | **WAF**; DDoS → **Shield** |
| Threat detection | **GuardDuty**; vuln scan → **Inspector**; PII in S3 → **Macie** |
| Who made the API call | **CloudTrail**; config history → **Config**; metrics/logs → **CloudWatch** |
| Restrict accounts in org | **SCP** |
| Multi-account SSO | **IAM Identity Center** |
| Rotate DB secrets | **Secrets Manager** |
| Dedicated key hardware | **CloudHSM** |
