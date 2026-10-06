# 24 — Security & Encryption (KMS, SSM, Secrets, ACM, CloudHSM, WAF, Shield, GuardDuty…)

## Why Encrypt?

| Type | How |
|---|---|
| **In flight** (TLS/SSL) | Encrypted before send, decrypted after receive → no **MITM** |
| **Server-side at rest** | Server encrypts after receiving, decrypts before sending with a **data key**; server must have key access |
| **Client-side** | Client encrypts, server **never** decrypts; receiving client decrypts. Can use **envelope encryption** |

## AWS KMS

"Encryption" for an AWS service usually = KMS. AWS manages keys; **IAM authorization**; **audit via CloudTrail**; integrated with EBS, S3, RDS, SSM… Also via API (SDK/CLI). **Never store secrets in plaintext** (encrypted secrets OK in code/env vars).

### Key types

| | Symmetric | Asymmetric |
|---|---|---|
| Algorithm | AES-256 (one key encrypts + decrypts) | RSA & ECC key pairs |
| Use | AWS-integrated services; you never see the key unencrypted (call KMS API) | Encrypt/Decrypt or **Sign/Verify**; public key downloadable, private key never. For users **outside AWS who can't call KMS API** |

| Key ownership | Cost |
|---|---|
| **AWS Owned Keys** (SSE-S3, SSE-SQS, SSE-DDB) | Free |
| **AWS Managed Keys** (`aws/rds`, `aws/ebs`) | Free |
| **Customer managed** (created in KMS) | **$1/month** |
| **Customer managed (imported)** | $1/month |
| API calls | $0.03 per 10,000 |

**Rotation**: AWS-managed → automatic every **1 year**; customer-managed → must enable (automatic + on-demand); **imported → manual only** (use alias).

### Key Policies
Control access to KMS keys (like bucket policies) — **you cannot control access without them**.
- **Default**: created if you don't provide one; **root user (entire account)** has complete access
- **Custom**: define who can use and who can administer; **cross-account access**

### Copying snapshots
- **Across regions**: KMS keys are regional → snapshot **re-encrypted with a key in the target region** (KMS ReEncrypt)
- **Across accounts**: snapshot encrypted with **customer managed key** → attach **key policy** authorizing target account → share snapshot → target **copies and re-encrypts with its own CMK** → create volume

### Multi-Region Keys
Identical keys (same **key ID**, key material, rotation) in different regions — **primary + replicas**, managed independently, **NOT global**. Encrypt in one region, decrypt in another with **no re-encrypt / no cross-region calls**.
Use: global client-side encryption, **DynamoDB Global Tables** (encrypt attributes with DynamoDB Encryption Client), **Aurora Global** (AWS Encryption SDK) — protects fields even from DB admins.

### S3 replication + KMS
- Unencrypted and **SSE-S3** replicate by default; **SSE-C** can replicate
- **SSE-KMS**: enable option, choose target KMS key, adapt key policy, IAM role with **`kms:Decrypt` (source) + `kms:Encrypt` (target)**; may hit KMS throttling → Service Quotas
- Multi-region keys are treated as **independent keys** by S3 (decrypt then encrypt)

### Sharing encrypted AMIs
Modify image attribute (**launch permission** for target account) → share the **KMS key** → target IAM needs `DescribeKey`, `ReEncrypt*`, `CreateGrant`, `Decrypt` → target may re-encrypt volumes with own key at launch.

## SSM Parameter Store

Secure **config + secrets** storage; optional **KMS** encryption; serverless, scalable, durable; **version tracking**; IAM security; EventBridge notifications; CloudFormation integration. **Hierarchy** (`/my-department/my-app/dev/db-url`) via `GetParameters` / `GetParametersByPath`. Can reference Secrets Manager secrets (`/aws/reference/secretsmanager/...`) and public AMI params.

| | Standard | Advanced |
|---|---|---|
| Parameters per account/region | 10,000 | 100,000 |
| Max value size | 4 KB | 8 KB |
| Parameter policies | No | **Yes** (TTL/expiration, ExpirationNotification, NoChangeNotification via EventBridge) |
| Cost | Free | $0.05/param/month |

## Secrets Manager

Newer service for **secrets**. **Force rotation every X days**, auto-generate on rotation (**Lambda**), **RDS integration** (MySQL, PostgreSQL, Aurora), **KMS** encrypted. **Multi-Region secrets**: replicas kept in sync, can **promote replica** to standalone (multi-region apps, DR).

## ACM (AWS Certificate Manager)

Provision/manage/deploy **TLS certificates** (public + private). **Public certs free**, **auto-renew** (60 days before expiry for ACM-issued). Load onto **ELB (CLB/ALB/NLB), CloudFront, API Gateway** (not directly on EC2).

- **Request**: FQDN (`corp.example.com`) or wildcard (`*.example.com`) → validation **DNS (preferred, CNAME in Route 53)** or **Email** (WHOIS contacts) → takes a few hours
- **Imported certs**: **no auto-renewal**; ACM emits daily expiry events starting **45 days** before (configurable) → EventBridge; Config rule `acm-certificate-expiration-check`
- **ALB**: HTTP→HTTPS redirect rule
- **API Gateway**: Edge-optimized → cert in **us-east-1**; Regional → cert in API's region; then CNAME/A-Alias in Route 53

## CloudHSM

**AWS provisions hardware, you manage keys** (KMS = AWS manages software). Dedicated, **tamper-resistant HSM**, **FIPS 140-2 Level 3**, symmetric + asymmetric, **no free tier**, need **CloudHSM client software**. IAM only for cluster CRUD; you manage keys/users. **Multi-AZ** cluster for HA. Redshift supports it; good with **SSE-C**. Integrates with KMS through a **Custom Key Store**.

| | KMS | CloudHSM |
|---|---|---|
| Tenancy | Multi-tenant | **Single-tenant** |
| Keys | AWS owned / managed / customer managed | **Customer managed** |
| Key types | Symmetric, asymmetric, signing | + **hashing** |
| Access | IAM | **You create users** |
| Availability | Multi-region access (can't use keys outside region created) | In VPC, share via VPC peering |
| Acceleration | None | **SSL/TLS, Oracle TDE** |
| HA | AWS managed | Add HSMs across AZs |
| Free tier | Yes | No |

## WAF (Web Application Firewall)

**Layer 7 (HTTP)** protection vs common exploits (**SQL injection, XSS**). Deploy on **ALB, API Gateway, CloudFront, AppSync GraphQL, Cognito User Pool**. **Not on NLB** (Layer 4).

- **Web ACL** rules: **IP sets (up to 10,000 IPs)**, HTTP headers/body/URI, size constraints, **geo-match**, **rate-based rules** (DDoS); **rule groups** reusable
- Web ACL is **regional** except for **CloudFront**
- **Fixed IP + WAF**: **Global Accelerator** (fixed IP) in front of **ALB** with WAF (WebACL in same region as ALB)

## Shield (DDoS)

| | Standard | Advanced |
|---|---|---|
| Cost | **Free**, all customers | **$3,000/month per organization** |
| Protects | **Layer 3/4**: SYN/UDP floods, reflection | EC2, ELB, CloudFront, Global Accelerator, Route 53 — sophisticated attacks |
| Extras | — | **24/7 DDoS Response Team (SRT)**, protection against **usage-spike fees**, **automatic layer 7 mitigation** (creates WAF rules) |

## Firewall Manager

Manage rules across **all accounts in an Organization**: **WAF rules**, **Shield Advanced**, **Security Groups** (EC2, ALB, ENI), **Network Firewall** (VPC), **Route 53 Resolver DNS Firewall**. **Regional** policies; auto-applies to **new resources and future accounts**.

**WAF vs Firewall Manager vs Shield**: used together. WAF alone for granular protection; **Firewall Manager** for cross-account/auto-protection of new resources; **Shield Advanced** adds SRT + advanced reporting for frequent DDoS.

## DDoS Resiliency Best Practices

| Layer | Practices |
|---|---|
| **Edge mitigation** | **CloudFront** (BP1), **Global Accelerator** (BP1), **Route 53** (BP3) |
| **Infrastructure** | ELB scales with traffic (BP6), **EC2 Auto Scaling** (BP7), Global Accelerator/Route 53/CloudFront |
| **Application** | CloudFront caches static content; **WAF** on CloudFront/ALB; **rate-based rules** auto-block bad IPs; managed rules (IP reputation, anonymous IPs); CloudFront geo-block; **Shield Advanced** auto L7 mitigation |
| **Attack surface reduction** | Hide resources behind CloudFront/API Gateway/ELB; **SGs + NACLs**; Elastic IPs protected by Shield Advanced; protect API endpoints (edge-optimized or CloudFront + regional; WAF + API Gateway burst limits, header filtering, API keys) |

## GuardDuty

**Intelligent threat discovery** — ML, anomaly detection, 3rd-party data. One click, 30-day trial, **no software to install**.

Inputs: **CloudTrail event logs** (unusual API calls, unauthorized deployments), **CloudTrail management events**, **CloudTrail S3 data events**, **VPC Flow Logs** (unusual internal traffic/IP), **DNS logs** (compromised EC2 sending encoded data in DNS queries); optional: EKS audit logs, RDS & Aurora, EBS, Lambda, S3.
Findings → **EventBridge** → Lambda/SNS. Dedicated **cryptocurrency attack** finding.

## Inspector

**Automated vulnerability assessment** — only **EC2 instances** (via **SSM agent**), **container images** (pushed to **ECR**), **Lambda functions** (code + dependencies). Continuous scanning; **package vulnerabilities (CVE database)**; **network reachability (EC2)**; **risk score** for prioritization. Reports to **Security Hub**, events to **EventBridge**.

## Macie

Managed **data security/privacy** with ML + pattern matching to **discover sensitive data (PII) in S3**; alerts via **EventBridge**.

## Exam Hints

- Audit encryption key usage → **KMS + CloudTrail**
- Need dedicated hardware / full key control / FIPS Level 3 → **CloudHSM**
- Secret rotation for RDS → **Secrets Manager**; cheap config/secret → **Parameter Store**
- Block SQL injection / XSS / geo / rate limit → **WAF**; NLB can't use WAF
- DDoS → **Shield** (+ CloudFront, Route 53); org-wide rules → **Firewall Manager**
- Threat detection from logs (CloudTrail, VPC Flow, DNS) → **GuardDuty**
- Vulnerability scanning EC2/ECR/Lambda → **Inspector**
- PII in S3 → **Macie**
- Free public TLS cert for ELB/CloudFront → **ACM**
