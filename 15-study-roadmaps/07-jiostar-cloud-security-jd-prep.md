# JioStar Cloud/Infra Security — JD-Mapped Prep

Role: **Cloud/Infrastructure Security Engineer, JioStar** (large-scale OTT streaming). The JD is a strong match for your real work — **lead with your SOC 2 / security experience**, and shore up the **OTT-specific** angles (CDN/DRM/abuse) and **DevSecOps tool names**, which are the likely differentiators.

---

## 1) Lead with YOUR experience (it maps almost line-for-line)
| JD asks for | Your real proof (say these) |
|---|---|
| Cloud security reviews, find misconfigs, remediate | **SOC 2 S3 audit tool** (boto3, scanned ACLs/bucket policies across **100+ buckets**, remediated public-access risks) |
| Audit/compliance (ISO 27001, SOC 2) | Same SOC 2 readiness work + evidence prep |
| IAM, access control, encryption | Least-privilege IAM, **reusable Terraform modules** (IAM/VPC/LB/DNS), **IRSA** |
| Automation (Python/Bash) for security | boto3 audit tool, **cross-account auto-tagging** (EventBridge/Lambda/CloudTrail), LB target-group isolation automation |
| Secrets / key management | **Zero-downtime secret rotation** (Secrets Manager + Lambda) across 30+ services |
| IaC (Terraform/CFN) | Terraform modules standardizing 28 accounts, **CloudFormation migration** |
| Monitoring/logging, SIEM | **CloudWatch + Kibana dashboards** correlating deploys with CI events |
| Incident response, triage, RCA | **On-call for 30+ services across 4 EKS clusters**, led RCAs, shipped corrective fixes |
| Harden containers | EKS hardening, **Istio sidecar** work, NetworkPolicies |

**Opener:** "I've done hands-on cloud security for **SOC 2** — public-access remediation across 100+ S3 buckets, zero-downtime secret rotation, least-privilege IAM, and cross-account governance tagging — plus production on-call and RCAs across 4 EKS clusters."

---

## 2) What to read (existing notes, takeaways only)
- `14-previous-interview-questions/03-jio-cloud-security.md` — real cloud-security Q&A (this IS Jio).
- `10-aws/118-cloud-security-fundamentals.md` — defense-in-depth + built-in tools.
- `10-aws/28-cloud-security-scenarios.md` — secrets-in-image, bucket+IP, egress, **CDN origin bypass**.
- `10-aws/29-authentication-vs-authorization.md` — the "how is it secured?" backbone.
- `08-kubernetes/40-cluster-hardening.md` — CIS/kube-bench/PSS/NetworkPolicies.
- `05-cicd/17-cicd-from-scratch-gitops-deep-dive.md` — where SAST/DAST/SCA gates fit.

---

## 3) Multi-cloud equivalents (JD = AWS **+ GCP + Azure**)
You're AWS-deep; say "AWS primarily, and I map the concepts across clouds." Know this table:
| Concept | AWS | GCP | Azure |
|---|---|---|---|
| Identity | IAM | Cloud IAM | Entra ID (Azure AD) |
| Firewall at instance | Security Group | VPC firewall rules | Network Security Group (NSG) |
| WAF | AWS WAF | Cloud Armor | Azure WAF (App Gateway/Front Door) |
| Threat detection | GuardDuty | Security Command Center | Microsoft Defender for Cloud |
| Audit log | CloudTrail | Cloud Audit Logs | Azure Activity Log |
| Secrets | Secrets Manager | Secret Manager | Key Vault |
| Keys/KMS | KMS | Cloud KMS | Key Vault |
| Config/compliance | Config / Security Hub | Security Health Analytics | Defender/Policy |
| Workload identity | IRSA / instance profile | **Workload Identity** | Managed Identity |

---

## 4) DevSecOps — shift-left (a named JD focus: SAST, DAST, SCA)
Know **what each is + a tool + where it runs**, as **blocking CI gates**:
- **SAST** (static — scans source) → SonarQube, Semgrep, CodeQL. Runs on code pre-build.
- **DAST** (dynamic — scans the running app) → OWASP ZAP, Burp. Runs against a deployed/staging instance.
- **SCA** (dependencies/CVEs in libraries) → Snyk, Dependabot, Trivy, OWASP Dependency-Check.
- **IaC scanning** → **Checkov / tfsec / Terrascan** (Terraform/CFN misconfig).
- **Secrets scanning** → gitleaks / trufflehog (block secrets in commits).
- **Image scanning** → Trivy / Grype / ECR scan.
- **The point:** wire them as **pipeline gates that FAIL the build** on high-severity findings — not advisory. Add **policy-as-code (OPA/Sentinel)** on Terraform plans.
- One-liner: "Shift-left: SAST on code, SCA on deps, IaC + secret scanning pre-merge, image scan pre-push, DAST on staging — all **blocking gates**, with policy-as-code on IaC."

---

## 5) SIEM, logging & incident response (JD-named)
- **Log sources → SIEM:** CloudTrail (API audit), GuardDuty (threats), Config (compliance/drift), **VPC Flow Logs**, ALB/CloudFront logs, EKS audit logs → centralized **SIEM** (Splunk / ELK / Microsoft Sentinel / Google Chronicle) for correlation + alerting.
- **Vuln management:** Inspector (EC2/ECR/Lambda), Trivy, continuous scanning + prioritized remediation (CVSS + exploitability).
- **Incident response flow (say this):** **detect → declare/triage (severity) → contain (isolate SG/revoke keys/quarantine) → eradicate → recover → blameless postmortem.** Gather logs (CloudTrail/Flow Logs), build a timeline, report. *(Note: `09-observability/14-incident-management-and-p1.md`.)*
- Your hook: on-call + RCAs across 4 EKS clusters; CloudWatch/Kibana triage dashboards.

---

## 6) Hardening — containers, VMs, OS (JD-named)
- **Kubernetes/Docker:** CIS Benchmark via **kube-bench**, **Pod Security Standards (restricted)**, **default-deny NetworkPolicies**, **Kyverno/OPA** admission, non-root + read-only rootfs + drop capabilities, image scanning, **restrict IMDS** (hop limit / block from pods), private API endpoint, encrypt etcd.
- **VM/OS:** CIS OS benchmarks, **golden/hardened AMIs** (Packer), patch management, disable root SSH / key-only / no password auth, minimal packages, host firewall, CloudWatch agent + file-integrity monitoring, **SSM instead of SSH**.
- **Baseline standards:** codify as **policy-as-code + hardened base images** so it's enforced, not documented-and-ignored.

---

## 7) Network segmentation, access, encryption (JD-named)
- **Segmentation:** private subnets for app/DB, public only for ALB/NAT/bastion; separate VPCs/accounts per env; **SG (stateful) vs NACL (stateless)**; **VPC endpoints/PrivateLink** to keep traffic off the internet.
- **Access:** least-privilege IAM, **no long-lived keys** → **OIDC federation → STS temp creds**, **IRSA** for pods, MFA, permission boundaries/SCPs.
- **Encryption:** at rest = **KMS** (EBS/S3/RDS, rotate keys); in transit = **TLS 1.2/1.3**; enforce TLS-only (`aws:SecureTransport`).

---

## 8) OTT-SPECIFIC security (your differentiator — JD calls out CDN, DRM, abuse)
This is where JioStar will test fit. You likely haven't done it — **learn the vocabulary and the controls** so you can speak to it intelligently.

**CDN security** (streaming is CDN-heavy):
- **Lock the origin to the CDN** (origin shielding, SG to CloudFront prefix list, **OAC** for S3, secret origin header) so no one bypasses the WAF by hitting the origin directly.
- **Signed URLs / signed cookies** + **token authentication** so only authorized/paying users fetch segments; short expiry.
- **WAF + rate limiting + bot control** at the edge; **AWS Shield / DDoS protection**; **geo-restriction** for licensing.
- TLS everywhere; cache-poisoning protections.

**Content protection / DRM:**
- **DRM systems:** **Widevine** (Google), **FairPlay** (Apple), **PlayReady** (Microsoft) — multi-DRM to cover all devices.
- Content is **encrypted** (AES-128 / CENC); the player gets a **decryption key from a license server** only after **authentication + entitlement** checks; **key rotation**; secure key delivery.
- **Forensic watermarking** to trace leaks back to an account; anti-piracy takedowns.

**Platform abuse** (OTT-specific threats):
- **Account takeover / credential stuffing** → MFA, rate limiting, bot detection, device fingerprinting, anomaly detection, breached-password checks.
- **Account/password sharing** → **concurrent-stream limits**, device limits, household/geo heuristics.
- **Scraping / catalog abuse / scalping** → WAF bot control, API rate limits, CAPTCHA.
- **Payment/subscription fraud** → fraud scoring, velocity checks.
- One-liner: "For OTT I'd protect the **content path** (lock origin to CDN, signed URLs, multi-DRM with entitlement checks, watermarking) and the **user path** (MFA, bot/WAF rate limiting, concurrency limits, anomaly detection for ATO and sharing)."

---

## 9) Compliance (JD: ISO 27001, SOC 2)
- **SOC 2** = trust-service criteria (security, availability, confidentiality…), evidence that controls operate — your audit tool directly supported this.
- **ISO 27001** = an **ISMS** (information security management system) + Annex A controls; risk-based.
- Your angle: "I prepared **evidence for SOC 2** — public-access remediation reports, IAM/least-privilege, change control, logging retention — and automated the scanning that produced that evidence."

---

## One-liners to rehearse
- **Opener:** SOC 2 cloud security — 100+ bucket public-access remediation, secret rotation, least-privilege IAM, cross-account tagging, EKS on-call/RCAs.
- **DevSecOps:** SAST (code) + DAST (running app) + SCA (deps) + IaC/secret/image scans — **blocking gates** + policy-as-code.
- **Multi-cloud:** "AWS deep; concepts map — SG↔NSG↔firewall rules, WAF↔Cloud Armor↔Azure WAF, GuardDuty↔SCC↔Defender."
- **Keyless:** OIDC → STS temp creds; IRSA for pods; no long-lived keys.
- **Bucket+IP:** not safe — private + IAM + pre-signed URL + Block Public Access.
- **CDN:** lock origin to the CDN (OAC/prefix-list/secret header) + signed URLs + WAF; a WAF only protects traffic through it.
- **DRM:** encrypted content + license server issues keys after entitlement; Widevine/FairPlay/PlayReady; watermarking.
- **Abuse:** MFA + bot/WAF rate limiting + concurrency limits + anomaly detection.
- **IR:** detect → triage → contain (isolate SG/revoke keys) → eradicate → recover → blameless postmortem; logs from CloudTrail/Flow Logs/GuardDuty → SIEM.
- **Honesty:** "I'm AWS-deep and OTT security is newer to me — here's how I'd reason about it…" (then CDN/DRM/abuse controls).
