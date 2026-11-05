<!-- #genai: AWS/cloud security scenario Q&A - secrets in images, egress to unknown host (preventive+detective), CDN origin bypass, IaC scanning gates, bucket public-vs-IP -->

## Question
Rapid-fire cloud-security scenarios: Is baking a cloud password into a server image OK? A bucket is "public but restricted to the office IP" — safe? An app suddenly sends traffic to an unknown internet server — name one **preventive** and one **detective** control. If an attacker reaches the **origin behind a CDN** directly, is the firewall still protecting you?

### Short Explanation
These test **secure-by-design instincts**: never embed static credentials, IP allow-listing is a weak control, you need **both prevention and detection** for exfiltration, and a **CDN/WAF only protects traffic that goes through it** — a reachable origin bypasses it.

## Answer (scenario by scenario)

### 1. Cloud password baked into a server startup image
- **Possible? Yes. Acceptable? No.** Anyone who pulls the image (or a leaked layer) gets the credential; it's long-lived and hard to rotate.
- **Right way:** attach an **identity to the workload** — AWS **instance profile / IRSA**, GCP **service account / Workload Identity** — so it gets **temporary, auto-rotated** credentials with no secret in the image. If a static secret is unavoidable, pull it at runtime from **Secrets Manager / Secret Manager / Vault**, never bake it in.

### 2. Bucket "public but restricted to the office IP" — safe?
- **Not really safe, and it's the wrong model.** IP allow-listing is a **coarse** control: no authentication, the office IP is shared by everyone on that network (and VPN/NAT users), and it's one policy edit away from fully public. There's **no identity** behind the access.
- **Right way:** keep the bucket **private (Block Public Access on)**, grant access via **IAM/identity** (roles, least privilege), hand out **pre-signed URLs** for temporary object access, and reach it privately over a **VPC endpoint**. Use IP conditions only as an *extra* layer, never the *only* one.

### 3. App suddenly sends traffic to an unknown internet server — one preventive + one detective
- **Preventive:** restrict **egress** — default-deny outbound via **security groups / NACLs / Kubernetes NetworkPolicies**, or force traffic through an **egress proxy/firewall with an allow-list** (and no public route / NAT-only). The unknown destination is simply blocked.
- **Detective:** **VPC Flow Logs + GuardDuty** (flags traffic to known-bad/unusual destinations, crypto-mining, C2), plus **DNS query logging** and anomaly alerts. You get an alert on the unexpected connection.
- (Bonus: this pattern is the classic **data-exfiltration / compromised-dependency** signal.)

### 4. Attacker connects directly to the origin behind the CDN — is the firewall protecting you?
- **No.** A **WAF/firewall attached to the CDN only inspects traffic that flows through the CDN.** If the origin is publicly reachable and the attacker hits it directly, they **bypass the CDN and its WAF entirely**.
- **Right way — lock the origin to the CDN only:**
  - Restrict the origin's **security group** to the **CDN's IP ranges** (CloudFront has an AWS-managed **prefix list**).
  - Require a **shared secret header** that the CDN injects and the origin validates (reject requests without it).
  - Use **CloudFront Origin Access Control (OAC)** for S3 origins; keep the origin off public DNS where possible.
  - Net effect: the only path to the origin is *through* the CDN/WAF.

## Detailed Explanation

### IaC misconfig scanning (related question: "tools that scan config files and block")
- Scan **IaC** (Terraform/CloudFormation/K8s manifests) with **Checkov, tfsec, Terrascan, KICS, or Trivy config** in CI.
- To **actually block**: run the scan as a **failing pipeline gate** on high-severity findings, and enforce **policy-as-code (OPA/Sentinel)** on the plan. Reporting alone doesn't stop a misconfig from shipping.

### Built-in cloud security tools worth naming (AWS)
**GuardDuty** (threat detection), **Security Hub** (aggregated findings/benchmarks), **Config** (compliance/drift), **Inspector** (vuln scanning), **Macie** (sensitive-data discovery in S3), **IAM Access Analyzer**, **CloudTrail** (audit). GCP equivalent: **Security Command Center**.

### The through-line
> Every scenario reduces to: **identity over static secrets**, **least privilege over coarse IP rules**, **prevent *and* detect**, and **a control only protects what actually passes through it**.

## Summary
- **No baked-in credentials** — use workload identity (instance profile/IRSA/Workload Identity) or a secrets store; temporary + rotated.
- **Public+office-IP bucket is not safe** — go **private + IAM + pre-signed URLs**; IP is a weak extra layer only.
- **Exfiltration**: **preventive = egress allow-list/default-deny**, **detective = Flow Logs + GuardDuty/DNS logging**.
- **CDN/WAF only protects traffic through it** — **lock the origin to the CDN** (prefix-list SG, secret header, OAC) or it's bypassable.
- Scan IaC (**Checkov/tfsec/Trivy**) as a **blocking** gate, not just a report.

## Related
- [S3 buckets & security](./113-s3-buckets-security.md) · [IP whitelisting](./117-ip-whitelisting.md)
- [Cloud security fundamentals](./118-cloud-security-fundamentals.md)
- [IRSA](./104-irsa.md) · [Kubernetes cluster hardening](../08-kubernetes/40-cluster-hardening.md)
