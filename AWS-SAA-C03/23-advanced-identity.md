# 23 — Advanced Identity in AWS

## AWS Organizations

**Global.** Manage **multiple AWS accounts**: one **management account** + **member accounts** (a member belongs to **one** organization only). Hierarchy: **Root OU → OUs → accounts** (by business unit, environment lifecycle, or project).

Benefits:
- **Consolidated billing**, one payment method
- **Aggregated usage discounts** (EC2, S3 volume), **shared Reserved Instances & Savings Plans** across accounts
- **API to automate account creation**
- Multi-account vs one-account-multi-VPC: better isolation
- Tagging standards for billing; **CloudTrail on all accounts → central S3 account**; **CloudWatch Logs → central logging account**
- **Cross-account roles** for admin

### SCPs (Service Control Policies)
- IAM-style policies applied to **OU or account** to restrict users and roles
- **Do NOT apply to the management account** (full admin)
- Deny by default model: need an **explicit allow from root through every OU in the path** to the account (default `FullAWSAccess` is attached)
- Explicit **Deny** in a parent OU wins (e.g. Sandbox OU denies S3 → all accounts below can't use S3)
- Strategies: **blocklist** (FullAWSAccess + Deny) and **allowlist**

### Tag Policies
Standardize tags: define tag **keys and allowed values**; helps cost allocation and **ABAC**; prevent non-compliant tagging (no effect on untagged resources); compliance report; **EventBridge** for non-compliance.

## IAM Conditions

| Condition key | Use |
|---|---|
| `aws:SourceIp` | Restrict client IP making API calls |
| `aws:RequestedRegion` | Restrict region of API calls |
| `ec2:ResourceTag` | Restrict by resource tags |
| `aws:MultiFactorAuthPresent` | Force MFA |
| **`aws:PrincipalOrgID`** | In **resource policies**: only accounts in your Organization |

**IAM for S3**: `s3:ListBucket` → bucket ARN (`arn:aws:s3:::test`) — bucket-level. `s3:GetObject/PutObject/DeleteObject` → object ARN (`arn:aws:s3:::test/*`) — object-level.

## IAM Roles vs Resource-Based Policies (cross-account)

| | Assume a Role | Resource-based policy |
|---|---|---|
| Permissions | **Give up original permissions**, take the role's | **Keep original permissions** |
| Example | — | User in A scans DynamoDB (A) and dumps to S3 in B → use S3 bucket policy |
| Supported by | — | S3, SNS, SQS, etc. |

**EventBridge security**: rule needs permission on target — **resource-based policy** (Lambda, SNS, SQS, S3, API Gateway) or **IAM role** (EC2 Auto Scaling, SSM Run Command, ECS task).

## IAM Permission Boundaries

- Supported for **users and roles (not groups)**
- A **managed policy** sets the **maximum** permissions an entity can get; effective = **intersection** of boundary and identity policy
- Combine with **Organizations SCP**
- Uses: delegate to non-admins (e.g. create IAM users), let developers self-assign policies without **privilege escalation**, restrict a **single user** (SCP restricts whole account)

**Policy evaluation**: explicit **Deny** wins → SCP must allow → resource policy → permission boundary → session policy → identity policy.

## IAM Identity Center (successor to AWS SSO)

**One login** for: all **AWS accounts in Organizations**, business cloud apps (Salesforce, Box, Microsoft 365), **SAML 2.0** apps, **EC2 Windows instances**.

- Identity providers: **built-in identity store**, or 3rd party **AD, OneLogin, Okta**
- **Permission Sets** = collection of IAM policies assigned to users/groups to define AWS access (e.g. ReadOnly for Dev, Full for Prod)
- Application assignments (SAML: provide URLs/certs/metadata)
- **ABAC**: fine-grained permissions from user attributes (cost center, title, locale) — define permissions once, change access by changing attributes

## Microsoft Active Directory & AWS Directory Services

AD on Windows Server (AD Domain Services): database of users, computers, printers, file shares, groups; centralized security; objects in **trees**, group of trees = **forest**.

| Directory Service | What | Notes |
|---|---|---|
| **AWS Managed Microsoft AD** | Your own AD in AWS | Manage users locally, **MFA**, **trust** with on-prem AD |
| **AD Connector** | **Proxy** to on-prem AD | Users managed on-prem; MFA |
| **Simple AD** | AD-compatible managed directory | **Cannot join on-prem AD** |

**Identity Center + AD**: AWS Managed Microsoft AD = out of the box. Self-managed directory = **two-way trust** with Managed AD, or **AD Connector**.

## AWS Control Tower

Easy set-up and governance of a secure, compliant **multi-account** environment (best practices) using **Organizations**. Automates setup, ongoing policy management with **guardrails**, detects violations/remediates, compliance dashboard.

| Guardrail | Implemented with | Example |
|---|---|---|
| **Preventive** | **SCPs** | Restrict regions across all accounts |
| **Detective** | **AWS Config** | Identify untagged resources (→ SNS / Lambda remediation) |

## Exam Hints

- Restrict what accounts can do (even root of member) → **SCP** (not for management account)
- Restrict one IAM user's max permissions → **Permission Boundary**
- Only org members access my bucket → `aws:PrincipalOrgID`
- SSO across accounts + SaaS apps → **IAM Identity Center**
- Proxy to on-prem AD → **AD Connector**; own AD in AWS → **Managed Microsoft AD**
- Set up multi-account landing zone with guardrails → **Control Tower**
