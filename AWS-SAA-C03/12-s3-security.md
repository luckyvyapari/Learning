# 12 — Amazon S3 Security

## Object Encryption

| Method | Keys | Header | Notes |
|---|---|---|---|
| **SSE-S3** | Owned/managed by AWS, **AES-256** | `x-amz-server-side-encryption: AES256` | **Default** for new buckets/objects |
| **SSE-KMS** | KMS keys | `x-amz-server-side-encryption: aws:kms` | User control + **audit via CloudTrail** |
| **DSSE-KMS** | KMS (layer 1) + S3 managed (layer 2) | `aws:kms:dsse` | Compliance needing 2 layers; higher cost/latency |
| **SSE-C** | **Customer-provided**, outside AWS; S3 **doesn't store the key** | Key in HTTP headers **every request** | **HTTPS mandatory** |
| **Client-side** | Customer encrypts/decrypts before/after S3 | — | Customer fully manages keys + cycle (S3 Client-Side Encryption Library) |

**SSE-KMS limit**: upload calls `GenerateDataKey`, download calls `Decrypt` → counts toward **KMS API quota** (5,500 / 10,000 / 30,000 req/s depending on region). Request increase via **Service Quotas**.

## Encryption in Transit

- HTTP endpoint (unencrypted) vs **HTTPS endpoint** (encrypted); HTTPS mandatory for SSE-C
- **Force HTTPS**: bucket policy with condition `aws:SecureTransport`
- **Force encryption at upload**: bucket policy denying PUT without encryption headers. Bucket policies are evaluated **before** default encryption

## CORS

Origin = scheme + host + port. Browser blocks cross-origin requests unless the other origin returns **CORS headers** (`Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`) — after a **preflight OPTIONS** request.

S3 case: web page in bucket A loads assets from bucket B → bucket B needs a CORS config allowing A's origin (or `*`). **Popular exam question.**

## MFA Delete

- Requires MFA to: **permanently delete an object version**, **suspend versioning**
- Not required to: enable versioning, list deleted versions
- **Versioning must be enabled**; only the **bucket owner (root)** can enable/disable MFA Delete

## Access Logs

Log **all requests** (authorized or denied, any account) to **another bucket in the same region**. **Never log into the monitored bucket** — creates a logging loop and exponential growth.

## Pre-Signed URLs

Generated via console, CLI, SDK. Holder **inherits permissions of the generator** for GET/PUT.

| Tool | Expiry |
|---|---|
| Console | 1 min – 720 min (12 h) |
| CLI | `--expires-in` seconds, default 3600, max 604,800 (168 h) |

Use: premium video for logged-in users, dynamic list of downloaders, temporary upload to a specific location.

## WORM Locks

| | Glacier **Vault Lock** | S3 **Object Lock** |
|---|---|---|
| Model | WORM; lock the vault policy so it can't be changed/deleted | WORM per **object version**; **versioning required** |
| Modes | — | **Compliance**: no one (even root) can overwrite/delete, retention can't be shortened. **Governance**: most users can't, privileged users can change |
| Extras | Compliance/retention | **Retention period** (extendable) · **Legal Hold** (indefinite, independent; `s3:PutObjectLegalHold`) |

## Access Points

Simplify security at scale: each access point has its own **DNS name** + **access point policy**, scoped to a prefix (e.g. `/finance`, `/sales`) instead of one giant bucket policy.
- **VPC Origin**: accessible only from inside the VPC; needs a **VPC Endpoint** (gateway or interface) whose policy allows both the bucket and access point.

## S3 Object Lambda

Lambda modifies an object **before returned** to the caller. One bucket + **Access Point** + **Object Lambda Access Point**.
Use cases: **redact PII**, convert XML → JSON, resize/watermark images per caller, enrich with database data.

## Exam Hints

- Audit key usage → **SSE-KMS** (CloudTrail)
- Customer holds the key, AWS doesn't store → **SSE-C** (HTTPS)
- Cross-origin browser error → **CORS**
- Prevent deletes/overwrites for compliance → **Object Lock (Compliance)**
- Temporary access to private object → **Pre-signed URL**
- Per-team access to one bucket → **Access Points**
