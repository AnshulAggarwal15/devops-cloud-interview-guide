# Jio — Cloud Security Interview

**Round style:** rapid-fire **cloud security** scenarios (AWS/GCP), several phrased as "yes/no and why" or "name a preventive and a detective control." The interviewer probed for **secure-by-design instincts**, not tool trivia.

> Each item: **"As asked"** (cleaned), **"What they're really asking"**, a short answer, and a link to the full write-up. A couple of the transcript answers were slightly off — those are flagged with the correction.

---

### Q1 — Which built-in cloud security tools have you used in production?
**As asked:** "Have you set up cloud access permissions for AWS/GCP? Which **built-in cloud security tool** have you actually used in **production**?"

**Short answer:** Name real ones and what they do: **GuardDuty** (threat detection), **Security Hub** (aggregated findings/CIS benchmarks), **Config** (compliance/drift), **Inspector** (vuln scanning), **Macie** (PII discovery in S3), **IAM Access Analyzer**, **CloudTrail** (audit). GCP: **Security Command Center**.

**Backed by:** [Cloud security fundamentals](../10-aws/118-cloud-security-fundamentals.md)

### Q2 — Keep sensitive servers off the public internet
**As asked:** "How do you set up the cloud network so **sensitive servers can't be reached from the public internet**?"

**Short answer:** Put them in **private subnets** (no route to an IGW); expose only what's needed via an **ALB/CDN** in public subnets; outbound via **NAT**; tighten **security groups/NACLs**; reach AWS services over **VPC endpoints**.

**Backed by:** [VPC & subnet design](../10-aws/116-vpc-and-subnet-design.md) · [VPC, subnets & routing](../10-aws/101-vpc-subnets-routing.md)

### Q3 — Developer wants to bake a cloud password into a server startup image
**As asked (paraphrased):**
> "A developer wants to put a **cloud password directly inside a server's startup image** so it can read from Google Storage. Is that possible? Is it right? What's the better way?"

**What they're really asking:** do you reject **static embedded credentials** and reach for **workload identity**?

**Short answer:** Possible, but **wrong** — anyone who pulls the image (or a cached layer) gets a long-lived credential that's hard to rotate. **Better:** attach an **identity to the workload** — GCP **service account / Workload Identity**, AWS **instance profile / IRSA** — for **temporary, auto-rotated** creds with **nothing baked in**. If a static secret is unavoidable, fetch it at runtime from **Secret Manager / Secrets Manager / Vault**.

**Backed by:** [Cloud security scenarios — secrets in images](../10-aws/28-cloud-security-scenarios.md) · [IRSA](../10-aws/104-irsa.md)

### Q4 — Have you hardened a Kubernetes cluster, and with which tool?
**As asked:** "What do you mean by a **hardened Kubernetes cluster**? Have you hardened one, and **which tool** did you use?"

**Short answer:** Hardening = least-privilege + defense-in-depth against the **CIS Kubernetes Benchmark**: disable anonymous auth, **RBAC** least privilege, audit logs, **etcd encryption**, **Pod Security `restricted`**, **default-deny NetworkPolicies**, image scanning, restrict IMDS. **Tools:** **kube-bench** (CIS), **kube-hunter**, **Trivy** (images/IaC), **Kyverno/OPA Gatekeeper** (admission blocking), **Falco** (runtime).

**Backed by:** [Kubernetes cluster hardening](../08-kubernetes/40-cluster-hardening.md)

### Q5 — Bucket "open to the world but restricted to the office IP" — safe?
**As asked (paraphrased):**
> "A cloud storage bucket is technically **open to the world but restricted to the office IP address**. Is the bucket **safe — yes or no, and why**? What's the right way to fix it?"

**Transcript note:** the answer given ("no — open to public, all data is public") reached the right verdict but the **reasoning was off**. It's *not* fully public if an IP condition is enforced — the real point is that **IP allow-listing is a weak control**.

**Short answer:** **Not safe (as a model).** IP allow-listing has **no identity/authentication**, the office IP is shared by everyone on that network (and VPN/NAT), and it's one edit from fully public. **Right way:** keep the bucket **private (Block Public Access on)**, grant access by **IAM/identity** (least privilege), use **pre-signed URLs** for temporary object access, and reach it via a **VPC endpoint**. IP conditions are an *extra* layer, never the only one.

**Backed by:** [S3 buckets & security](../10-aws/113-s3-buckets-security.md) · [IP whitelisting](../10-aws/117-ip-whitelisting.md)

### Q6 — Tools that scan infra config files for security issues — and *block*
**As asked:** "Have you used tools that **scan infrastructure configuration files** for security issues and **actually block** them?"

**Short answer:** Yes — **Checkov / tfsec / Terrascan / KICS / Trivy config** scan Terraform/CloudFormation/K8s manifests. To make them **block**, run them as a **failing CI gate** on high-severity findings and enforce **policy-as-code (OPA/Sentinel)** on the plan, plus **admission control (Kyverno/Gatekeeper)** at deploy time. Reporting alone doesn't stop a misconfig from shipping.

**Backed by:** [Cloud security scenarios — IaC scanning](../10-aws/28-cloud-security-scenarios.md) · [Cloud security fundamentals](../10-aws/118-cloud-security-fundamentals.md)

### Q7 — App suddenly sends traffic to an unknown internet server — one preventive, one detective
**As asked (paraphrased):**
> "A startup app suddenly starts sending traffic **out to an unknown server on the internet**. Name **two ways this could have been caught earlier: one preventive and one detective**."

**Short answer:**
- **Preventive:** restrict **egress** — default-deny outbound (**security groups / NACLs / NetworkPolicies**) or force traffic through an **egress proxy/firewall allow-list**. The unknown destination is blocked.
- **Detective:** **VPC Flow Logs + GuardDuty** (flags unusual/known-bad destinations, C2, crypto-mining) and **DNS query logging** with anomaly alerts.
- (This is the classic **exfiltration / compromised-dependency** signal.)

**Backed by:** [Cloud security scenarios — egress](../10-aws/28-cloud-security-scenarios.md)

### Q8 — Attacker reaches the origin behind the CDN directly — is the firewall still protecting you?
**As asked (paraphrased):**
> "You front the app with a **CDN** (and firewall/WAF). If an attacker finds the **actual server behind the CDN** and connects **directly**, is the firewall still protecting you?"

**Short answer:** **No.** A WAF/firewall attached to the CDN only inspects traffic **through** the CDN — hitting the origin directly **bypasses** it. **Fix — lock the origin to the CDN only:** restrict the origin's **security group to the CDN's IP prefix list** (e.g. CloudFront managed prefix list), require a **secret header** the CDN injects and the origin validates, use **Origin Access Control (OAC)** for S3, and keep the origin off public DNS. The only path to the origin should be *through* the CDN/WAF.

**Backed by:** [Cloud security scenarios — CDN origin bypass](../10-aws/28-cloud-security-scenarios.md)

---

## The through-line (say this if they ask for principles)
Every scenario reduces to four instincts:
1. **Identity over static secrets** — workload identity, not baked-in passwords.
2. **Least privilege over coarse IP rules** — private + IAM, not public + IP filter.
3. **Prevent *and* detect** — egress control *and* Flow Logs/GuardDuty.
4. **A control only protects what passes through it** — a reachable origin bypasses the CDN/WAF.

## One-line recap to rehearse
- **Secrets:** "Never bake credentials into an image — use **workload identity** (Workload Identity / IRSA) for temporary, rotated creds."
- **Bucket:** "**Private + IAM + pre-signed URLs**; IP allow-listing is a weak extra layer, not security."
- **Hardening:** "**CIS Benchmark via kube-bench**, enforce with **Kyverno + blocking CI gates**, watch runtime with **Falco**."
- **Exfiltration:** "**Preventive = egress allow-list**, **detective = Flow Logs + GuardDuty**."
- **CDN:** "WAF only protects CDN traffic — **lock the origin to the CDN** (prefix-list SG, secret header, OAC)."
