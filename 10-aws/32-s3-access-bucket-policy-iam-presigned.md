<!-- #genai: AWS interview note - granting access to a specific S3 object; bucket policy vs IAM policy vs pre-signed URL; cross-account and temporary access -->

## S3 Access — Bucket Policy vs IAM vs Pre-signed URL (and Cross-Account)

### Question
How would you give a user from **another account/organization** access to a **specific file** in an S3 bucket? What's the difference between an **S3 bucket policy, an IAM policy, and a pre-signed URL** — and how do you give **temporary** access to a single object?

### Short Explanation
Three different tools for "who can touch this object": **IAM policy** (attached to *your* identities — "what can this principal do"), **bucket policy** (attached to the *bucket* — "who can touch this bucket", and the go-to for **cross-account**), and **pre-signed URL** (a **time-limited, credential-free link** to one object — the go-to for **temporary** or **anonymous** access). Pick by *who* needs access and *for how long*.

## Answer
- **Give a specific file to a user in another account:** grant it with a **bucket policy** that allows that account's principal `s3:GetObject` on the **specific object ARN** (`.../path/file.txt`), and the other account must also allow their user to call S3 (both sides needed). For a single fetch, a **pre-signed URL** is simpler.
- **Bucket policy vs IAM vs pre-signed URL:** IAM = identity-based (your users/roles); bucket policy = resource-based (great for **cross-account** and public/anonymous rules); pre-signed URL = **temporary, signed** access to **one object** for someone with **no AWS credentials**.
- **Temporary access to one object:** generate a **pre-signed URL** with a short **expiry** — anyone with the link can GET (or PUT) that object until it expires, no IAM identity required.

## Detailed Explanation

### The three mechanisms
| | IAM policy | Bucket policy | Pre-signed URL |
|---|---|---|---|
| Attached to | An **identity** (user/role) | The **bucket** (resource) | Nothing — it's a **signed URL** |
| Answers | "What can this principal do?" | "Who can touch this bucket?" | "Here's temporary access to **one object**" |
| Cross-account | Needs both sides | **Grants another account directly** | Works for anyone with the link |
| Best for | Your own users/workloads | Cross-account, org-wide, public rules | **Temporary / no-credential** access to a single object |
| Lifetime | Until changed | Until changed | **Expires** (seconds–hours) |

Effective permission = the union of applicable allows, and **any explicit `Deny` wins**; **Block Public Access** overrides permissive rules.

### Give another account access to a specific object (bucket policy)
```json
{
  "Effect": "Allow",
  "Principal": { "AWS": "arn:aws:iam::222222222222:user/partner" },
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::my-bucket/reports/2026/file.txt"
}
```
- Scope `Resource` to the **exact object ARN** (not the whole bucket) for least privilege.
- The **other account** must also grant *their* user permission to call `s3:GetObject` (cross-account needs an allow on **both** sides).
- Cleaner at scale: have the partner **assume a role** in your account (`sts:AssumeRole`) scoped to that prefix, instead of naming their user in the bucket policy.

### Pre-signed URL (temporary access to one object)
A pre-signed URL embeds a **signature from an existing principal's credentials** plus an **expiry**, letting someone **without AWS access** GET/PUT that specific object until it expires.
```bash
aws s3 presign s3://my-bucket/reports/2026/file.txt --expires-in 3600   # 1 hour
```
- Great for: a download link for an external user, a browser **upload** (presigned PUT), sharing a file without creating IAM users.
- The URL inherits the **generator's permissions**, scoped to that object, and **auto-expires** — no long-lived grant.

### Which to choose
- **Another AWS account, ongoing:** **bucket policy** (or cross-account **AssumeRole**).
- **Someone with no AWS identity / one-off / time-boxed:** **pre-signed URL**.
- **Your own users/apps:** **IAM policy** (and IRSA for pods).

### ⚠️ Things they probe
- **Cross-account = bucket policy (or AssumeRole)**; **temporary/no-creds = pre-signed URL**; **your identities = IAM**.
- Scope to the **exact object ARN** for least privilege; keep **Block Public Access on**.
- Cross-account needs an allow on **both** sides.
- Pre-signed URLs **expire** and carry the **signer's** permissions — set a short TTL.

## Summary
- **IAM policy** = what *your* principals can do; **bucket policy** = resource-based, the tool for **cross-account/org** grants; **pre-signed URL** = **time-limited signed link** to **one object** for someone with no AWS creds.
- Give another account a specific file via a **bucket policy on the exact object ARN** (both sides must allow) or a **pre-signed URL** for a one-off.
- **Temporary single-object access → pre-signed URL with a short expiry.** Keep least privilege + Block Public Access on.

## Related
- [S3 buckets & security](./113-s3-buckets-security.md) · [IP whitelisting](./117-ip-whitelisting.md)
- [Lambda connect to S3 in another account](./17-lambda-connect-s3-other-account.md) · [AWS STS](./18-aws-sts.md)
- [Authentication vs Authorization framework](./29-authentication-vs-authorization.md)
