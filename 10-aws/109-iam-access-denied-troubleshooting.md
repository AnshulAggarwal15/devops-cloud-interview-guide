<!-- #genai: AWS interview note - troubleshooting IAM AccessDenied, a common R2 scenario for the Docusign cloud round -->

## Question
A request is failing with `AccessDenied` (or a pod/CI job can't call an AWS API). How do you troubleshoot it?

### Short Explanation
Work outward from the identity: confirm **who** is making the call, read the **denial message**, then check the policy chain — remembering that an **explicit deny always wins** and boundaries (SCP/permission boundary) can silently block an allow.

## Answer
1. Identify the caller: `aws sts get-caller-identity` (which user/role/session is it *actually*?).
2. Read the error — it often names the **action** and **resource** denied.
3. Check the identity's **permission policy** — is the `Action` + `Resource` allowed?
4. Check for an **explicit Deny** anywhere (deny beats allow).
5. Check **boundaries**: SCPs (org), permission boundaries, session policies.
6. Check **resource-based policy** (e.g. S3 bucket policy, KMS key policy) and any `Condition` (IP, MFA, VPC).

## Detailed Explanation

### The usual culprits
- **Wrong identity**: the pod/instance assumed a different role than you think -> `get-caller-identity` first. On EKS, is **IRSA** wired (ServiceAccount annotation + role trust on `namespace:serviceaccount`)?
- **Missing action/resource**: policy allows `s3:GetObject` on `bucket/*` but not `s3:ListBucket` on `bucket`; or the ARN/path doesn't match.
- **Explicit deny / SCP**: an org SCP or permission boundary blocks it regardless of the identity's allow.
- **Resource policy**: the bucket/KMS key policy doesn't grant the caller (cross-account especially).
- **KMS**: reading an encrypted secret/object also needs `kms:Decrypt` on the key.
- **Condition failed**: policy requires MFA, a source IP range, or a VPC endpoint.

### Tools to name
- **IAM Policy Simulator** and **`--dry-run`** to test.
- **CloudTrail** to see the exact denied call, the principal, and often the reason.

## Summary
- Start with **`aws sts get-caller-identity`** — verify the real principal.
- Trace the chain: identity policy -> **explicit deny** -> SCP/boundary -> resource policy -> KMS -> conditions.
- Common causes: wrong assumed role (IRSA misconfig), missing action/resource, SCP deny, missing `kms:Decrypt`. Use CloudTrail + Policy Simulator.
