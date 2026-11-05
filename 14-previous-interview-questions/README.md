# 14 - Previous Interview Questions

Real questions from actual interviews, reconstructed from transcripts, then cleaned up and mapped to detailed answer notes elsewhere in this repo.

## Why this section exists
The hardest part of these rounds is usually **not** the concept — it's the way the question is **phrased** (long, indirect, layered follow-ups). Each file here:
- keeps a **"how it was actually asked"** version so you can practice parsing convoluted wording, and
- gives a **crisp "what they're really asking"** plus a short answer and a **link to the full write-up**.

## Companies
| # | Company | Focus |
|---|---|---|
| 1 | [Nielsen](./01-nielsen.md) | IDP + Terraform safety, Terraform meta-args/providers/state, Kubernetes autoscaling (HPA, custom metrics, KEDA) |
| 2 | [Amagi](./02-amagi.md) | **R1:** cross-ecosystem auth (OIDC/IRSA), CoreDNS internals & scale, pod IAM/IMDS, multi-cluster observability, Argo CD/GitOps, API gateways/JWT · **R2:** cross-region latency/keep-alive, HTTP/1.1-2-3 + multiplexing, TLS handshake & 1.2-vs-1.3, Terraform drift |
| 3 | [Jio](./03-jio-cloud-security.md) | Cloud security: workload identity vs secrets, bucket exposure, cluster hardening, IaC scanning, egress/exfiltration, CDN origin protection |
| 4 | [MoveInSync](./04-moveinsync.md) | CI/CD-from-scratch + GitOps drilled per-step (authN/authZ every hop), Jenkins agents/triggers, EKS-from-scratch/Terraform, app→K8s migration, CIDR, RBAC — plus interviewer-conduct notes |

## Question banks (not company-specific)
| File | Use it for |
|---|---|
| [Common AWS DevOps/Cloud/SRE questions](./05-common-aws-devops-sre-questions.md) | The "everyone gets asked these" set for 3–5 yrs — a coverage-mapped **self-audit checklist** linking each question to its answer note |
| [20 real interview-level questions](./06-twenty-real-interview-questions.md) | A widely-shared 20-question set (CI/CD, K8s scheduling, Terraform, GitOps, service mesh, scaling, incident) — coverage-mapped for **timed drilling** |
| [K8s / Terraform / Helm / AWS / Docker set](./07-k8s-terraform-helm-aws-docker-bank.md) | Scenario set heavy on K8s (kube-proxy, services, probes, cordon/drain), plus Terraform drift/state, Prometheus/Grafana, Helm, S3 access, Docker |

## How to use
1. Read the "as asked" line and try to answer **before** reading the crisp version.
2. Open the linked answer note for the full explanation.
3. Rehearse the one-liner takeaway from each answer note out loud.
