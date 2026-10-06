# 02 — IAM (Identity & Access Management)

**Global service.** Controls *who* can do *what* on *which* resources.

## Building Blocks

| Concept | Meaning |
|---|---|
| **Root account** | Created by default. Use only for account setup. Never share |
| **User** | One physical person = one IAM user |
| **Group** | Contains **users only** (no nested groups). A user can be in many groups |
| **Policy** | JSON document defining permissions |
| **Role** | Permissions for AWS services/resources (EC2, Lambda, CloudFormation) |

Principle: **least privilege** — grant only what's needed.

## Policy Structure

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "optional-id",
    "Effect": "Allow",
    "Principal": "account/user/role",
    "Action": ["s3:GetObject"],
    "Resource": "arn:aws:s3:::my-bucket/*",
    "Condition": {}
  }]
}
```

| Field | Required? | Meaning |
|---|---|---|
| Version | Yes | Always `2012-10-17` |
| Id | No | Policy identifier |
| Statement | Yes | One or more statements |
| Sid | No | Statement id |
| Effect | Yes | `Allow` / `Deny` |
| Principal | For resource-based | Who the policy applies to |
| Action | Yes | API calls allowed/denied |
| Resource | Yes | Resources affected |
| Condition | No | When the policy applies |

Inheritance: users get policies from their groups + inline policies attached directly.

## Password Policy & MFA

- Password policy: min length, character types, allow self-change, expiration, prevent reuse
- **MFA = password (know) + device (own)** — stolen password alone ≠ compromise
- MFA options: virtual (Google Authenticator, Authy), **U2F security key** (YubiKey), hardware key fob (Gemalto), GovCloud fob (SurePassID)

## Access Methods

| Method | Protected by |
|---|---|
| Management Console | Password + MFA |
| CLI | Access keys |
| SDK (code) | Access keys |

- Access Key ID ≈ username, Secret Access Key ≈ password. **Never share.**
- **CLI**: open source tool, direct access to public APIs, scriptable
- **SDK**: language libraries (JS, Python, Java, Go, .NET, mobile, IoT); CLI is built on the Python SDK

## IAM Roles for Services

AWS service performs actions on your behalf → attach a **role** (not access keys).
Common: EC2 instance roles, Lambda function roles, CloudFormation roles.

## Security Tools

| Tool | Scope | Shows |
|---|---|---|
| **Credentials Report** | Account-level | All users + status of their credentials |
| **Access Advisor** | User-level | Services granted + last accessed → trim policies |

## Best Practices

- Don't use root except for setup
- 1 physical user = 1 IAM user
- Permissions on **groups**, not users
- Strong password policy + enforce MFA
- Roles for services; access keys only for CLI/SDK
- Audit with Credentials Report + Access Advisor
- Never share users/keys

## Exam Hints

- Root account → only account setup tasks
- EC2 needs AWS access → **IAM Role**, never embed keys
- Find unused permissions → **Access Advisor**; find stale credentials → **Credentials Report**
