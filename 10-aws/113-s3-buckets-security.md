<!-- #genai: AWS interview note - S3 buckets, storage classes, encryption, bucket policy vs IAM, public access, versioning for the cloud round -->

## Question
Explain Amazon S3 — storage classes, how you secure a bucket, and how access is decided.

### Short Explanation
S3 is **object storage** (buckets + objects, 11 nines of durability). The interview usually splits into two parts: **cost** (choosing the right **storage class** + lifecycle) and **security** (block public access, encryption, and **bucket policy vs IAM**).

## Answer
- **Storage classes** trade retrieval speed/cost: **Standard** for hot data, **Intelligent-Tiering** when access is unpredictable, **Standard-IA/One Zone-IA** for infrequent, **Glacier/Deep Archive** for cold archive.
- **Security**: keep **Block Public Access ON**, encrypt at rest (**SSE-S3 / SSE-KMS**), enforce **TLS in transit**, and grant access via **IAM policies** (identity) or **bucket policies** (resource).
- **Versioning + MFA delete + Object Lock** protect against accidental/malicious deletion.

## Detailed Explanation

### Storage classes (cost lever)
| Class | Best for |
|---|---|
| **Standard** | Frequently accessed, low-latency |
| **Intelligent-Tiering** | Unknown/changing access patterns (auto-moves tiers) |
| **Standard-IA** | Infrequent access, multi-AZ |
| **One Zone-IA** | Infrequent + re-creatable (single AZ, cheaper) |
| **Glacier Instant / Flexible / Deep Archive** | Archival; minutes-to-hours retrieval, lowest cost |

Use **lifecycle rules** to transition objects (e.g. Standard → IA after 30 days → Glacier after 90) and expire old versions.

### Access control — bucket policy vs IAM
| | IAM policy | Bucket policy |
|---|---|---|
| Attached to | **Identity** (user/role) | **Resource** (the bucket) |
| Answers | “what can this principal do?” | “who can touch this bucket?” |
| Cross-account | Needs both sides | Can grant another account directly |

Effective permission = union of all `Allow`s, and **any explicit `Deny` wins**. **Block Public Access** overrides permissive policies/ACLs — leave it on unless you truly need public objects (then prefer CloudFront + OAC over a public bucket).

### Encryption
- **At rest**: SSE-S3 (S3-managed keys) or **SSE-KMS** (audited, per-key control); default encryption can be enforced on the bucket.
- **In transit**: enforce HTTPS with a bucket policy condition `aws:SecureTransport = false → Deny`.

### Data protection
- **Versioning** keeps every version → recover overwrites/deletes.
- **MFA Delete** and **Object Lock (WORM)** stop tampering/ransomware.
- **Replication (CRR/SRR)** for DR/compliance; **VPC gateway endpoint** to reach S3 without the internet.

### Things they probe
- ACLs are legacy — prefer **policies** and disable ACLs (bucket owner enforced).
- A “public bucket” incident is almost always **Block Public Access off + a wildcard policy** — know how to spot it.
- S3 is **regional** but the namespace is **global** (bucket names are globally unique).

## Summary
- Pick the **storage class** by access pattern; automate transitions with **lifecycle rules**.
- Secure with **Block Public Access on, KMS encryption, TLS-only, least-privilege IAM/bucket policy** (explicit deny wins).
- Protect data with **versioning, Object Lock, replication**; reach S3 privately via a **gateway endpoint**.

## Related
- [Core services, secrets & cost](./108-core-services-secrets-and-cost.md)
- [IP whitelisting](./117-ip-whitelisting.md)
- [Cloud security fundamentals](./118-cloud-security-fundamentals.md)
