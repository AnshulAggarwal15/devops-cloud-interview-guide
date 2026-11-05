<!-- #genai: AWS interview note - IP whitelisting with security groups, NACLs, ALB, WAF, prefix lists, S3 policy conditions for the cloud round -->

## Question
How do you implement IP whitelisting (allow-listing) on AWS, and at which layers?

### Short Explanation
"Whitelist these IPs" can mean several layers — **security group, NACL, ALB rules, WAF, or resource policy conditions**. Interviewers want you to pick the **right layer** and to know that IP allow-listing is a **coarse control**, not a substitute for authentication.

## Answer
- For instance/ENI-level access, allow the CIDRs in a **security group** (stateful, allow-only) — the most common answer.
- At the subnet edge, use **NACLs** (stateless, allow **and** deny) — useful to **block** specific IPs.
- For public web apps behind an **ALB/CloudFront**, use **AWS WAF** IP sets, or ALB listener rules, to allow/deny at L7.
- For AWS resources (S3, APIs), enforce via **policy conditions** (`aws:SourceIp` / `aws:VpcSourceIp`).

## Detailed Explanation

### Choose the layer
| Layer | Control | Use it for |
|---|---|---|
| **Security Group** | Stateful, **allow-only** | Allow office/VPN CIDRs to an EC2/RDS/ALB — default choice |
| **NACL** | Stateless, allow **+ deny** | **Block** a bad IP/range at the subnet boundary |
| **ALB listener rules** | L7 | Simple host/path + source conditions |
| **AWS WAF** (ALB/CloudFront/API GW) | L7, managed **IP sets** | Web allow/deny lists, rate limiting, geo, big/managed rule sets |
| **Resource policy condition** | IAM | `aws:SourceIp` on S3 buckets, API Gateway, KMS, etc. |

### Security group example (allow SSH from an office range)
```text
Type   Protocol  Port  Source
SSH    TCP       22    203.0.113.0/24   # office egress CIDR only
HTTPS  TCP       443   0.0.0.0/0        # public app
```
SGs are **stateful** — return traffic is auto-allowed; there is **no deny**, only allow. To deny, use a NACL.

### Manage many IPs cleanly
- Use **managed prefix lists** to keep one named list of CIDRs and reference it from multiple SGs/route tables — update in one place.
- In **WAF**, use an **IP set** referenced by rules; update the set without touching rules.

### S3 / API restriction by IP
```json
{
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:*",
  "Resource": "arn:aws:s3:::my-bucket/*",
  "Condition": { "NotIpAddress": { "aws:SourceIp": ["203.0.113.0/24"] } }
}
```
Deny unless the request comes from the allowed range (combine with Block Public Access + auth).

### Things they probe
- IP allow-listing is **coarse** and breaks with dynamic/residential IPs, NAT, and mobile users → pair it with **real authN/Z** (IAM, OIDC, mTLS).
- SG = **stateful allow-only**; NACL = **stateless, needs both inbound & outbound rules** and supports **deny**.
- For users behind proxies/CDNs, whitelist on the **X-Forwarded-For** at WAF, not the connecting IP.
- Prefer **VPN/PrivateLink/VPC endpoints** over public IP allow-lists where possible.

## Summary
- Pick the layer: **SG (allow-only, stateful)** for most cases, **NACL** to **deny**, **WAF/ALB** for L7 web apps, **policy conditions** (`aws:SourceIp`) for S3/APIs.
- Manage scale with **prefix lists** and **WAF IP sets**.
- IP allow-listing is a **coarse** guardrail — always combine with authentication and prefer private connectivity.

## Related
- [Security Groups vs NACLs](./102-security-groups-vs-nacls.md)
- [Cloud security fundamentals](./118-cloud-security-fundamentals.md)
- [S3 buckets & security](./113-s3-buckets-security.md)
