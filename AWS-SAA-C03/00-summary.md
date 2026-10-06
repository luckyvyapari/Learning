# AWS Certified Solutions Architect – Associate (SAA-C03) — Study Notes

Personal quick-reference notes following Stéphane Maarek's Udemy course. Each file = tables + key rules + exam hints. Read this page first, open a file when you need detail.

**Exam**: 65 questions · 130 min · pass 720/1000 · scenario-based · eliminate wrong answers · "feasible but complicated" is usually wrong.

## Index

| # | File | Covers |
|---|---|---|
| 01 | [getting-started](01-getting-started.md) | Regions, AZs, edge locations, global vs regional services |
| 02 | [iam](02-iam.md) | Users, groups, policies, roles, MFA, access keys, CLI/SDK |
| 03 | [ec2-basics](03-ec2-basics.md) | Instance types, security groups, user data, purchasing options, Spot |
| 04 | [ec2-associate](04-ec2-associate.md) | Elastic IP, placement groups, ENI, hibernate |
| 05 | [ec2-storage](05-ec2-storage.md) | EBS types, snapshots, AMI, instance store, EFS |
| 06 | [high-availability-scalability](06-high-availability-scalability.md) | ALB/NLB/GWLB, sticky sessions, SNI, ASG, scaling policies |
| 07 | [rds-aurora-elasticache](07-rds-aurora-elasticache.md) | RDS, replicas vs Multi-AZ, Aurora, RDS Proxy, ElastiCache |
| 08 | [route53](08-route53.md) | Records, alias, routing policies, health checks, hybrid DNS |
| 09 | [classic-solutions-architecture](09-classic-solutions-architecture.md) | Stateless/stateful web apps, WordPress, Elastic Beanstalk |
| 10 | [s3](10-s3.md) | Buckets, versioning, replication, storage classes |
| 11 | [s3-advanced](11-s3-advanced.md) | Lifecycle, events, performance, Batch Operations, Storage Lens |
| 12 | [s3-security](12-s3-security.md) | Encryption, CORS, MFA Delete, pre-signed URLs, Object Lock, Access Points |
| 13 | [cloudfront-global-accelerator](13-cloudfront-global-accelerator.md) | CDN, origins, OAC, Global Accelerator |
| 14 | [storage-extras](14-storage-extras.md) | Snow family, FSx, Storage Gateway, Transfer Family, DataSync |
| 15 | [integration-messaging](15-integration-messaging.md) | SQS, SNS, Kinesis, Firehose, Amazon MQ |
| 16 | [containers](16-containers.md) | Docker, ECS, Fargate, ECR, EKS |
| 17 | [serverless](17-serverless.md) | Lambda, edge functions, DynamoDB, API Gateway, Step Functions, Cognito |
| 18 | [serverless-architectures](18-serverless-architectures.md) | Mobile app, blog site, microservices, CloudFront offloading |
| 19 | [databases-in-aws](19-databases-in-aws.md) | Choosing a DB, DocumentDB, Neptune, Keyspaces, Timestream |
| 20 | [data-analytics](20-data-analytics.md) | Athena, Redshift, OpenSearch, EMR, QuickSight, Glue, Lake Formation, MSK |
| 21 | [machine-learning](21-machine-learning.md) | Rekognition, Transcribe, Polly, Lex, Comprehend, Kendra, Textract… |
| 22 | [monitoring-audit](22-monitoring-audit-cloudwatch-cloudtrail-config.md) | CloudWatch, EventBridge, CloudTrail, Config |
| 23 | [advanced-identity](23-advanced-identity.md) | Organizations, SCP, permission boundaries, Identity Center, AD, Control Tower |
| 24 | [security-encryption](24-security-encryption.md) | KMS, SSM Parameter Store, Secrets Manager, ACM, CloudHSM, WAF, Shield, GuardDuty |
| 25 | [vpc](25-vpc.md) | CIDR, subnets, NAT, NACL vs SG, peering, endpoints, VPN, Direct Connect, Transit Gateway, IPv6 |
| 26 | [disaster-recovery-migrations](26-disaster-recovery-migrations.md) | RPO/RTO, DR strategies, DMS/SCT, MGN, AWS Backup |
| 27 | [more-solutions-architecture](27-more-solutions-architecture.md) | HPC, HA EC2, CloudFormation, SSM, Batch, Outposts, cost tools |
| 28 | [well-architected-exam-tips](28-well-architected-exam-tips.md) | 6 pillars, Trusted Advisor, exam logistics, decision cheat sheet |
| 29 | [course-outline-map](29-course-outline-map.md) | All 33 course sections mapped to notes + hands-on-only extras (CloudShell, Budgets, ports) |

## Top 25 Numbers & Facts to Remember

| Topic | Fact |
|---|---|
| Regions/AZs | 3 AZs typical (min 3, max 6) |
| EC2 discounts | Reserved/Savings Plan up to **72%**, Convertible 66%, Spot up to **90%** |
| Spot | **2-min** warning; cancel request, then terminate |
| Security groups | Inbound blocked / outbound allowed by default; timeout = SG, refused = app |
| Placement groups | Spread max **7 per AZ** |
| EBS | AZ-locked; gp3 3,000 IOPS baseline; io2 Block Express **256,000 IOPS**; Multi-Attach **16** instances |
| ELB | NLB static IP/UDP/TCP; ALB path/host routing; GWLB port **6081** |
| ASG | Cooldown **300 s** |
| RDS | Replicas up to **15** (async) · Multi-AZ sync · backup **1–35 days** |
| Aurora | **6 copies/3 AZ**, 4/6 write, 3/6 read, failover < 30 s, storage to **256 TB** |
| Route 53 | CNAME not on apex; Alias free + works on apex; health check threshold 3, interval 30 s |
| S3 | Object max **50 TB**; multipart > **5 GB**; 3,500 write / 5,500 read per prefix; durability **11 9s** |
| S3 classes | Standard-IA / One Zone-IA min 30 d; Glacier IR/FR 90 d; Deep Archive 180 d |
| SQS | Retention 4 d (max 14 d); visibility **30 s**; long poll up to **20 s**; message **1,024 KB**; FIFO 300/3,000 msg/s |
| SNS | 12.5 M subs/topic, 100 K topics |
| Kinesis | Retention up to **365 d**; shard 1 MB/s in, 2 MB/s out |
| Lambda | **15 min**, 10 GB RAM, 1,000 concurrency, 50 MB zip / 250 MB unzipped |
| DynamoDB | Item **400 KB**; PITR **35 d**; DAX µs reads; Streams 24 h |
| CloudTrail | Retention **90 days**; data events off by default |
| KMS | Customer key **$1/mo**; auto-rotation 1 yr |
| Shield Advanced | **$3,000/mo** per org |
| VPC | 5 VPCs/region; CIDR **/16–/28**; **5 IPs reserved** per subnet |
| NAT Gateway | 5 → **100 Gbps**; one per AZ for HA |
| Direct Connect | Dedicated 1–400 Gbps; not encrypted by default |
| Snowball | 210 TB edge storage optimized; use if transfer > 1 week |

## Concept Map (what solves what)

| Problem | Reach for |
|---|---|
| **Compute** | EC2 (ASG + ELB) · Lambda · ECS/Fargate · EKS · Batch · Beanstalk |
| **Storage** | S3 (+classes) · EBS · EFS · FSx · Instance Store · Storage Gateway · Snow |
| **Database** | RDS/Aurora (SQL) · DynamoDB (NoSQL) · ElastiCache · DocumentDB · Neptune · Keyspaces · Timestream · Redshift (OLAP) · OpenSearch |
| **Decoupling** | SQS · SNS · Kinesis · EventBridge · MQ · Step Functions |
| **Networking** | VPC · IGW/NAT · SG/NACL · Peering · Transit GW · Endpoints/PrivateLink · VPN · Direct Connect · Route 53 · CloudFront · Global Accelerator |
| **Security** | IAM · Organizations/SCP · KMS · Secrets Manager · ACM · WAF · Shield · GuardDuty · Inspector · Macie · Cognito |
| **Observability/Audit** | CloudWatch · CloudTrail · Config · VPC Flow Logs |
| **Migration/DR** | DMS · SCT · MGN · DataSync · Snowball · AWS Backup · Elastic DR |
| **Analytics/ML** | Athena · Glue · EMR · QuickSight · Lake Formation · MSK · SageMaker + AI services |
| **Governance/Cost** | Control Tower · Cost Explorer · Anomaly Detection · Trusted Advisor · Well-Architected Tool |
