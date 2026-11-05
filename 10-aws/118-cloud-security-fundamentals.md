<!-- #genai: AWS interview note - cloud security fundamentals, shared responsibility, least privilege, encryption, defense in depth for the cloud round -->

## Question
How do you think about security on AWS — what's your layered approach?

### Short Explanation
This is a breadth question. Interviewers want a **structured, layered answer** (defense in depth) instead of one trick: the **shared responsibility model**, **identity/least privilege**, **network isolation**, **encryption**, **secrets**, and **detection/audit** — with a concrete example of each.

## Answer
- Start with the **shared responsibility model**: AWS secures the cloud; you secure what's **in** it (data, IAM, config, patching).
- Apply **defense in depth** across layers: **identity (least privilege) → network → data (encryption) → secrets → detection/audit**.
- Automate and continuously verify: **IaC guardrails, no long-lived keys, everything encrypted, everything logged**.

## Detailed Explanation

### 1. Shared responsibility
- **AWS**: physical, hypervisor, managed-service internals.
- **You**: IAM, security groups, OS patching (EC2), encryption choices, data classification, app security.

### 2. Identity — least privilege
- Prefer **roles over IAM users**; no long-lived access keys (use STS/IRSA/instance profiles).
- Grant **least privilege**, add **permission boundaries** and org-level **SCPs**, require **MFA**, rotate credentials.
- Evaluation rule: **explicit deny always wins** (see the IAM note).

### 3. Network isolation
- Private subnets for app/DB, public only for ALB/NAT/bastion.
- **SGs (stateful)** + **NACLs (stateless)**; **WAF** at the edge; **VPC endpoints/PrivateLink** to keep traffic off the internet.

### 4. Data protection — encryption everywhere
- **At rest**: KMS (EBS, S3, RDS, EFS). Control keys, rotate, audit usage via CloudTrail.
- **In transit**: TLS enforced (e.g. S3 `aws:SecureTransport`, ALB HTTPS listeners, mTLS internally).

### 5. Secrets
- **Secrets Manager** (rotation) / **Parameter Store** (config) — never hardcode.
- On EKS, sync with the **External Secrets Operator**; enable **etcd encryption at rest**.

### 6. Detection, audit & response
| Service | Role |
|---|---|
| **CloudTrail** | Audit log of every API call |
| **Config** | Resource compliance / drift rules |
| **GuardDuty** | Threat detection from logs |
| **Security Hub** | Aggregated findings / benchmarks |
| **Inspector** | Vulnerability scanning (EC2/ECR/Lambda) |
| **Macie** | Sensitive-data (PII) discovery in S3 |

### Things they probe
- Name the **most common breach cause**: over-permissive IAM and **public S3 buckets** → least privilege + Block Public Access.
- Shift left: scan IaC (tfsec/Checkov), images (Trivy), and gate in CI.
- Have an **incident story**: detect (GuardDuty) → contain (revoke keys/isolate SG) → eradicate → learn.

## Summary
- **Shared responsibility** first, then **defense in depth**: identity → network → data → secrets → detection.
- **Least privilege + no long-lived keys, encrypt everything, log everything (CloudTrail), detect with GuardDuty/Config/Security Hub.**
- Automate guardrails in IaC/CI and rehearse an incident-response flow.

## Related
- [IAM users, roles & policies](./103-iam-users-roles-policies.md)
- [Secrets Manager vs Parameter Store](./22-secrets-manager-vs-parameter-store.md)
- [IP whitelisting](./117-ip-whitelisting.md)
- [Vault — secrets management](./119-vault-secrets-management.md)
