# 29 — Course Outline → Notes Map

Maps every section of the Udemy course (33 sections) to the note file that covers it. **Hands-on** lectures are console walkthroughs — the concepts are in the notes, the clicking is practice only. **Video-only** items have no slide content, so the small extras are at the bottom of this file.

| # | Course section | Notes | Notes on coverage |
|---|---|---|---|
| 1 | Introduction | — | Admin: account creation, activation, instructor. Create account + enable MFA on root |
| 2 | Code & Slides Download | — | Admin |
| 3 | Getting Started with AWS | [01](01-getting-started.md) | Regions, AZs, console, global vs regional |
| 4 | IAM & AWS CLI | [02](02-iam.md) | Policies, MFA, keys, CLI/SDK, roles, tools. **CloudShell** → extras below |
| 5 | EC2 Fundamentals | [03](03-ec2-basics.md) | **AWS Budget setup** → extras below. Instance roles in [02](02-iam.md) |
| 6 | EC2 – SAA Level | [04](04-ec2-associate.md) | Elastic IP, placement groups, ENI, hibernate |
| 7 | EC2 Instance Storage | [05](05-ec2-storage.md) | EBS, snapshots, AMI, instance store, volume types, multi-attach, encryption, EFS |
| 8 | HA & Scalability: ELB & ASG | [06](06-high-availability-scalability.md) | ALB, NLB, GWLB, sticky, cross-zone, SSL/SNI, draining, ASG policies |
| 9 | RDS + Aurora + ElastiCache | [07](07-rds-aurora-elasticache.md) | **Ports list** → extras below |
| 10 | Route 53 | [08](08-route53.md) | All routing policies, health checks, resolver |
| 11 | Classic Solutions Architecture | [09](09-classic-solutions-architecture.md) | WhatsTheTime, MyClothes, WordPress, Beanstalk |
| 12 | S3 Introduction | [10](10-s3.md) | Buckets, policy, website, versioning, replication, classes, Express One Zone |
| 13 | Advanced S3 | [11](11-s3-advanced.md) | Lifecycle, analytics, requester pays, events, performance, Batch, Storage Lens |
| 14 | S3 Security | [12](12-s3-security.md) | Encryption (SSE-S3/KMS/DSSE/C), CORS, MFA Delete, logs, pre-signed, locks, access points, Object Lambda |
| 15 | CloudFront & Global Accelerator | [13](13-cloudfront-global-accelerator.md) | Origins, geo restriction, invalidation, GA |
| 16 | AWS Storage Extras | [14](14-storage-extras.md) | Snow, FSx, Storage Gateway, Transfer Family, DataSync, comparison |
| 17 | Decoupling: SQS, SNS, Kinesis, MQ | [15](15-integration-messaging.md) | All covered |
| 18 | Containers: ECS, Fargate, ECR, EKS | [16](16-containers.md) | All covered |
| 19 | Serverless Overviews | [17](17-serverless.md) | Lambda (limits, concurrency, SnapStart, @Edge, VPC), DynamoDB, API Gateway, Step Functions, Cognito |
| 20 | Serverless Architectures | [18](18-serverless-architectures.md) | MyTodoList, MyBlog, microservices, update offloading |
| 21 | Databases in AWS | [19](19-databases-in-aws.md) | Summaries + DocumentDB, Neptune, Keyspaces, Timestream |
| 22 | Data & Analytics | [20](20-data-analytics.md) | Athena, Redshift, OpenSearch, QuickSight, Glue, Lake Formation, Flink, MSK, pipeline |
| 23 | Machine Learning | [21](21-machine-learning.md) | All 11 services |
| 24 | Monitoring & Audit | [22](22-monitoring-audit-cloudwatch-cloudtrail-config.md) | **Logs Live Tail** → extras below |
| 25 | IAM – Advanced | [23](23-advanced-identity.md) | Organizations, SCP, tag policies, conditions, evaluation, Identity Center, Directory Services |
| 26 | Security & Encryption | [24](24-security-encryption.md) | KMS, SSM PS, Secrets Manager, ACM, CloudHSM, WAF, Shield, FM, GuardDuty, Inspector, Macie |
| 27 | Networking – VPC | [25](25-vpc.md) | CIDR → costs; Regional NAT GW; Network Firewall |
| 28 | Disaster Recovery & Migrations | [26](26-disaster-recovery-migrations.md) | DRS, DMS/SCT, RDS migrations, on-prem strategies, Backup, MGN, large transfers, VMware |
| 29 | More Solution Architectures | [27](27-more-solutions-architecture.md) | Event processing, caching, IP blocking, HPC, HA EC2 |
| 30 | Other Services | [27](27-more-solutions-architecture.md) | CloudFormation, SES, Pinpoint, SSM, Cost Explorer, Anomaly, Outposts, Batch, AppFlow, Amplify, Instance Scheduler |
| 31 | WhitePapers & Architectures | [28](28-well-architected-exam-tips.md) | Well-Architected, tool, Trusted Advisor |
| 32 | Preparing for the Exam | [28](28-well-architected-exam-tips.md) | Exam logistics + tips |
| 33 | Congratulations | [28](28-well-architected-exam-tips.md) | Certification paths |

## Extras (video / hands-on only topics — not in the slide deck)

### AWS CloudShell
- Browser-based shell launched from the console, **pre-authenticated** with your console credentials (no access keys to configure)
- **Free**; pre-installed AWS CLI, Python, Node.js, git etc.
- **Persistent storage** in the home directory (~1 GB per region); available only in **select regions** (check region availability)
- Files can be uploaded/downloaded; multiple tabs; can use your IAM permissions only

### AWS Budgets (set up on day 1)
- Create **cost / usage / reservation / savings-plan budgets** with **alerts** (email or SNS) at actual or forecasted thresholds
- Enable **IAM access to billing data** so non-root users can view costs; use **Cost Explorer** for analysis, **Budgets** for alerts, **Cost Anomaly Detection** for ML spikes
- Always set a **zero-spend / small budget** when practicing to avoid surprise bills; **delete resources** after labs (NAT Gateways, RDS, Elastic IPs, load balancers cost money)

### Default Ports to Know

| Port | Service |
|---|---|
| 21 | FTP |
| 22 | SSH / SFTP |
| 80 | HTTP |
| 443 | HTTPS |
| 1433 | Microsoft SQL Server |
| 1521 | Oracle |
| 3306 | MySQL / MariaDB / Aurora MySQL |
| 3389 | RDP (Windows) |
| 5432 | PostgreSQL / Aurora PostgreSQL |
| 6081 | GENEVE (Gateway Load Balancer) |
| 6379 | Redis (ElastiCache) |
| 11211 | Memcached (ElastiCache) |

### CloudWatch Logs Live Tail
Real-time streaming view of incoming log events in CloudWatch Logs (filter by log group/stream/pattern) — for live debugging, unlike **Logs Insights** which is a query engine over stored data.

### Other hands-on-only items
- **Console simultaneous sign-in**: use multiple browser profiles / the multi-session support to stay signed into several accounts
- **AWS Account Activation Troubleshooting**: payment method verification, wait up to 24 h
- **Exam extras**: 50% discount voucher after passing an AWS exam; **+30 min** accommodation for non-native English speakers (request before booking)
