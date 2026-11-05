<!-- #genai: AWS interview note - how resources communicate (inside VPC, cross-subnet/AZ/VPC/account), which services use Security Groups vs IAM, exceptions, and a scenario Q bank; for the Docusign R2 cloud round -->

## Question
How do AWS resources communicate — inside a VPC, across subnets/AZs/VPCs/accounts — and how do you reason about it?

### Short Explanation
Every AWS connection must satisfy **two independent layers**:
1. **Network reachability** — is there a *route* and do the *firewalls* (SG/NACL) allow it?
2. **Identity / permissions** — does **IAM** allow the action (for API-based services)?

A call fails if *either* is missing. Most "it can't connect" problems are one of these two.

---

## The mental model (say this first in an interview)
> "I separate every AWS communication into **network path** and **IAM permission**. For EC2↔EC2 or
> EC2↔RDS it's mostly network (routes + security groups). For EC2↔S3 or Lambda↔DynamoDB it's mostly IAM
> plus a network path to the service endpoint. If something can't talk, I check which of those two layers is
> broken."

---

## 1. Inside a VPC — resources just route to each other
- Every subnet has an implicit, unremovable route: **`10.0.0.0/16 → local`** (the VPC CIDR).
- So **any resource in any subnet/any AZ of the same VPC can reach any other by private IP** — no IGW, no NAT needed. A VPC **spans all AZs in the region**, so cross-AZ is the same as cross-subnet at the routing layer (it just costs cross-AZ data transfer).
- What actually *gates* the traffic:
  - **Security Group** (instance/ENI level, **stateful**) — the primary control.
  - **NACL** (subnet level, **stateless**) — secondary, rarely changed.
- **Best practice:** allow by **SG reference**, not IP. e.g. the RDS SG allows inbound `5432` *from the app SG* — not from a CIDR. Self-documenting and moves with autoscaling.

## 2. EC2 → S3 (the "service isn't in my VPC" case)
S3 is **not inside your VPC** — it's a regional service with a public API endpoint. So EC2 reaching S3 needs **both**:
- **A network path** — one of:
  - **Internet path:** public subnet (IGW) or private subnet via **NAT Gateway** → S3 public endpoint (traffic leaves AWS's edge).
  - **VPC Gateway Endpoint for S3 (preferred):** adds a route (`prefix-list → vpce`) so traffic to S3 stays on the **AWS private network** — no internet, no NAT, cheaper, more secure.
- **IAM permission** — the EC2 **instance role (instance profile)** must allow `s3:GetObject` etc., and the **bucket policy** must not deny. (If the object is KMS-encrypted, also `kms:Decrypt`.)

**So EC2→S3 can fail three ways:** no network path (no NAT/endpoint), no IAM permission, or a bucket/KMS policy denial. This is a *classic* interview trap — candidates give only the IAM answer and miss the network path.

## 3. Different subnet / AZ / VPC / account

| Boundary | How they talk | Gotchas |
|---|---|---|
| **Different subnet, same VPC** | Automatic via `local` route | Check SG + NACL only |
| **Different AZ, same VPC** | Same — VPC spans AZs | Works, but **cross-AZ data transfer costs** |
| **Different VPC** | **VPC Peering** (1:1), **Transit Gateway** (hub for many), or **PrivateLink** (expose one service) | **CIDRs must not overlap**; **peering is non-transitive**; update route tables *both* sides |
| **Different account** | Same network options (peering/TGW/PrivateLink all work cross-account) **+** IAM: cross-account **role assumption** or **resource policies** | Network *and* IAM both needed |
| **To on-prem** | **VPN** or **Direct Connect** (often via Transit Gateway) | — |

- **VPC Peering** — simple 1:1, non-transitive (A–B and B–C does **not** give A–C). SGs can reference peer SGs (same region).
- **Transit Gateway** — hub-and-spoke; connects many VPCs/accounts/on-prem; **transitive**. Use when you have more than a few VPCs.
- **PrivateLink (interface endpoint)** — expose a single service across VPCs/accounts without full network connectivity (great for SaaS/shared services).
- **Cross-account IAM** — the target role's **trust policy** names the other account; the caller needs `sts:AssumeRole`. Or use a **resource-based policy** (e.g. S3 bucket policy granting another account).

## 4. Which services use Security Groups vs IAM? (the key framework)
**Ask one question: "Does it have an ENI / private IP inside my VPC?"**

- **YES → it has a Security Group.** You control comms with SGs (and NACLs at the subnet):
  - **EC2**, **RDS/Aurora**, **ElastiCache**, **Redshift**, **ALB/NLB**, **EFS mount targets**,
    **EKS pods / ECS tasks** (they get ENIs), **Lambda-in-VPC**, **Interface VPC Endpoints (PrivateLink)**,
    **OpenSearch-in-VPC**.
- **NO → it's a regional "API" service reached over an endpoint → NO Security Group.** You control access with **IAM policies + resource policies (+ VPC endpoint policies)**:
  - **S3**, **DynamoDB**, **SQS**, **SNS**, **KMS**, **Secrets Manager / SSM Parameter Store**, **STS**,
    **CloudWatch**, **ECR** (the API), most `*.amazonaws.com` API services.

**Endpoint nuance:**
- **Gateway endpoints** (only **S3** and **DynamoDB**): route-table based, **free**, controlled by an **endpoint policy** — no SG.
- **Interface endpoints** (everything else via PrivateLink): backed by an **ENI**, so they **do** have an SG.

## 5. Most common communication patterns
- **App → App (in-VPC):** private IP / internal ALB / Cloud Map service discovery, gated by **SG-to-SG** rules.
- **App → managed API service (S3/DynamoDB/SQS/KMS):** **IAM role** + a path to the endpoint (VPC endpoint preferred). No SG on the service side.
- **External user → App:** **Route 53** → **public ALB** (public subnet) → **targets** (private subnet).
- **Cross-account/VPC:** peering/TGW/PrivateLink for the network + IAM roles/resource policies for permission.

## 6. Exceptions & why (great "do they really get it?" material)
- **S3 & DynamoDB aren't in your VPC** → no SG; access = IAM + (bucket/endpoint) policy. Reason: they're API services, not ENIs.
- **Lambda networking is opt-in.** By default Lambda runs in AWS-managed network *with* internet access. **Attach it to your VPC** and it gets ENIs + an SG but **loses default internet** (now needs a NAT/endpoint to reach S3 etc.). Reason: VPC attachment changes its egress path. This surprises people.
- **NACLs are stateless** → you must allow **return traffic** (ephemeral ports `1024–65535`); SGs (stateful) don't need this.
- **Peering is non-transitive** and needs **both** route tables updated.
- **Cross-AZ / cross-region traffic works transparently but costs money.**
- **"IAM allows it but it still fails"** → check the **network path** (no NAT/endpoint), a **resource-policy deny**, **KMS**, or a failed **Condition** (MFA/IP/VPC).

---

## Scenario Q bank (rehearse the checklist for each)
1. **Two EC2s in different subnets, same VPC, can't ping/connect.** → `local` route exists by default, so check **SG** (inbound rule referencing the other), then **NACL** (both directions), then app/port.
2. **EC2 in a private subnet can't `yum`/`apt` install.** → No outbound internet: missing **NAT Gateway** route (or a VPC endpoint for the repo).
3. **EC2 can't read S3 even though IAM allows it.** → No **network path** (no NAT/gateway endpoint), or **bucket policy** deny, or **KMS** `Decrypt` missing, or **VPC endpoint policy** too strict.
4. **Let an EKS pod write to S3 privately, no internet.** → **S3 Gateway Endpoint** (network) + **IRSA** (permission).
5. **Two VPCs are peered but still can't reach each other.** → **Route tables** not updated on both sides, **SG/NACL**, or **overlapping CIDRs**.
6. **Account A app must read a bucket in account B.** → B's **bucket policy** grants A (or A **assumes a role** in B); plus network if it's private-only.
7. **Lambda worked, then started timing out on RDS after I "put it in the VPC."** → RDS **SG** doesn't allow the Lambda **SG**, or the Lambda subnet has no route; and it may now lack internet for other calls.
8. **RDS unreachable from app tier.** → RDS **SG** must allow the **app SG** on the DB port; correct subnet/route; is RDS public or private?
9. **Need many VPCs across accounts to interconnect.** → **Transit Gateway** (transitive hub), not a mesh of peerings.
10. **Expose an internal service to another team's VPC without full connectivity.** → **PrivateLink / endpoint service**.

---

## Summary
- Every connection = **network path (routes + SG/NACL/endpoint)** **AND** **IAM (for API services)**. Diagnose which layer is broken.
- **Same VPC** = automatic `local` route across subnets/AZs; SG/NACL gate it.
- **EC2→S3** needs a path (NAT or **gateway endpoint**) **and** an IAM role (+ bucket/KMS policy).
- **Cross-VPC** = Peering (1:1, non-transitive) / Transit Gateway (hub) / PrivateLink; **cross-account** adds IAM roles/resource policies. CIDRs must not overlap.
- **SG applies only to things with an ENI in your VPC** (EC2, RDS, ALB, EKS pods, interface endpoints…). **API services (S3, DynamoDB, SQS, KMS…) have no SG — control them with IAM + resource/endpoint policies.**
