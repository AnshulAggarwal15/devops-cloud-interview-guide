# DevOps and Cloud Interview Guide

Practical notes, troubleshooting scenarios, and interview-preparation packs for DevOps, SRE, Cloud, and Platform Engineering roles.

## Start here

Choose the route that matches your goal:

1. **Broad technical preparation:** work through the topic index below in order, prioritizing Linux, networking, containers, cloud, Terraform, Kubernetes, delivery, and observability.
2. **Kubernetes:** start with the [Kubernetes one-shot handbook](./08-kubernetes/00-kubernetes-one-shot-handbook.md), then open the numbered deep dives only for topics that need more depth.
3. **AWS:** use the curated [AWS index](./10-aws/README.md).
4. **Time-boxed interview loop:** start with the [Docusign interview master plan](./docusign-interview-master-plan.md), then use the round-specific packs.

This repository currently begins at `02-git`; there is no `01` module. Some later sections are placeholders and are marked accordingly below.

## Topic index

| Area | Best entry point | Coverage/status |
|---|---|---|
| Git | [Most-used Git commands](./02-git/17-most-used-git-commands.md) | Commands, branching, merge/rebase, conflicts, recovery, tags, and secret removal |
| Linux | [Most-used Linux commands](./03-linux/01-most-used-linux-commands.md) | Operations, permissions, CPU/disk/network failures, service health, and shell exercises |
| Networking | [DNS](./04-networking/01-explain-dns.md) and [OSI/request flow](./04-networking/02-explain-osi-model-with-example.md) | DNS, proxies, subnets, connection failures, latency, and 502 troubleshooting |
| CI/CD | [Jenkins shared libraries](./05-cicd/01-jenkins-shared-libraries.md) | Scenario collection for Jenkins, artifacts, build failures, pipeline performance, Argo CD, and notifications; no broad overview yet |
| Terraform | [Terraform modules](./06-terraform/02-terraform-modules.md) | `for_each`, module design, imports, and state recovery; assumes basic Terraform knowledge |
| Docker | [Docker commands](./07-docker/11-docker-commands.md) | Runtime failures, networking, persistence, image behavior, registries, disk, and debugging |
| Kubernetes | [One-shot handbook](./08-kubernetes/00-kubernetes-one-shot-handbook.md) | Primary map plus 29 focused architecture, networking, delivery, and troubleshooting deep dives |
| Observability & SRE | [Section index](./09-observability/README.md) | Three pillars, Prometheus/Grafana, multi-cluster (Thanos/Mimir), SLI/SLO/SLA & error budgets, incident/P1 handling, latency/OOM troubleshooting |
| AWS | [AWS index](./10-aws/README.md) | VPC, IAM, EKS identity, load balancing, HA, autoscaling, services, cost, and troubleshooting |
| Azure | [Section README](./11-azure/README.md) | **Coming soon** — placeholder only |
| Python | [Conceptual questions](./12-python/04-python-conceptual-interview-questions.md) then [coding questions](./12-python/05-python-programming-interview-questions.md) | Packages, practical scripts, log parsing, concepts, and coding practice |
| Project stories | [Section README](./13-project-related/README.md) | **Coming soon** — placeholder only |
| Previous interview questions | [Section index](./14-previous-interview-questions/README.md) | Real, company-tagged questions from transcripts, cleaned up and mapped to detailed answer notes |
| Study roadmaps | [Section index](./15-study-roadmaps/README.md) | Personalized, gap-driven study plans from honest self-assessments (Kubernetes: 2–3 month mastery path + limited-time crash path) |

## Kubernetes reading path

Avoid rereading the same overview repeatedly:

- **Primary map:** [`00-kubernetes-one-shot-handbook.md`](./08-kubernetes/00-kubernetes-one-shot-handbook.md)
- **Architecture and scheduling:** files 01, 02, 06, 17, 18, 27, 28, and 29
- **Services, networking, and ingress:** files 03–12 and 21–24
- **Delivery and rollback:** files 13–16
- **Troubleshooting and runtime behavior:** files 19, 20, 25, and 26

The handbook intentionally provides the overview; the numbered files are deep dives. In particular, file 13 should be treated as the concise deployment-strategy answer and file 16 as the broader comparison.

## Interview packs

| Resource | Use it for |
|---|---|
| [Interview master plan](./docusign-interview-master-plan.md) | Main sequence and prioritization for the interview loop |
| [Cloud/DevOps 1-hour cram](./docusign-cloud-devops-1hr-cram.md) | Fast AWS, Kubernetes, delivery, security, and troubleshooting refresh |
| [Debug-order playbook](./docusign-debug-order-playbook.md) | Evidence-driven incident and troubleshooting answers |
| [Request lifecycle handbook](./docusign-request-lifecycle-handbook.md) | End-to-end DNS, load balancer, Kubernetes, application, and observability request flow |
| [Round 1 coding practice](./docusign-r1-coding-practice.md) | Timed scripting/coding exercises |
| [Round 2 coding/system-design cram](./docusign-r2-coding-systemdesign-cram.md) | Coding and architecture rehearsal |

## How to use these notes

For each important topic, do not stop at reading. Demonstrate that you can:

```text
explain → implement → break → diagnose → recover → discuss trade-offs
```

For scenario answers, use:

```text
symptoms → scope → recent changes → hypotheses → evidence
→ mitigation → root cause → permanent fix → prevention
```

Prefer one deeply understood lab, runbook, or incident story over ten memorized definitions.

## Repository boundaries

The `cc` directory contains uncurated Cursor conversation exports, including unrelated material. It is intentionally excluded from the learning path. Treat it as an archive/scratch area, not as authoritative interview guidance.

## Companion notes (personal `Learn` folder)

These live **outside this repository**, in a personal study folder (`../Learn/`), and are **not part of the course**. They complement these notes with deeper theory, cross-cutting production scenarios, and behavioral prep. The highest-value ones:

| File | Use it for |
|---|---|
| [DevOps interview master Q&A index](../Learn/33-devops-interview-master-index.md) | One compact Q&A bank mapped to this course, plus high-value gap questions (init containers, sidecars, CSI, ENI, EventBridge, Step Functions, service mesh) |
| [Production scenario answer bank](../Learn/34-production-scenario-answer-bank.md) | Reusable troubleshooting flows: pending pods, CrashLoop/OOM, node `NotReady`, service/ingress, Terraform partial apply, regional outage, DNS/TLS, latency |
| [AWS architecture study guide](../Learn/02-aws-architecture.md) | VPC, 3-tier design, ALB vs NLB, RDS Multi-AZ, S3 classes/lifecycle, IAM |
| [Observability study guide](../Learn/06-observability.md) | Three pillars, PromQL, SLI/SLO/SLA, error budgets, alerting, incident management |
| [Kubernetes hands-on guide](../Learn/03-kubernetes-hands-on.md) | Core objects, kubectl commands, troubleshooting, RBAC, cluster patterns |
| [CI/CD & deployment strategies](../Learn/11-cicd-deployment-strategies.md) | Rolling / blue-green / canary / A-B, feature flags, pipeline design |
| [Situational & behavioral questions](../Learn/15-situational-behavioral-questions.md) | STAR stories: conflict, mistakes, prioritization, "tell me about yourself", "where in 5 years" |

> These are personal companion notes — study aids, not authoritative course content.
