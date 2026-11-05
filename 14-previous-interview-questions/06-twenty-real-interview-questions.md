# 20 Real Interview-Level Questions — Coverage Map

A widely-shared set of 20 mid-senior DevOps questions. Each maps to its answer note. Use it as a **timed drill**: cover the right column, answer out loud in 60–90 seconds, open the note only when you stumble.

**Legend:** ✅ dedicated note · ⭐ added while filling this bank · ⚠️ answered inline below

| # | Question | Where |
|---|---|---|
| 1 | CI/CD end-to-end in production | ✅ [CI/CD from scratch + GitOps](../05-cicd/17-cicd-from-scratch-gitops-deep-dive.md) |
| 2 | Blue-Green vs Canary (+ when/example) | ✅ [Deployment strategies](../08-kubernetes/16-all-deployment-strategies.md) |
| 3 | How K8s schedules a Pod (control-plane process) | ⭐ [Pod scheduling lifecycle](../08-kubernetes/44-pod-scheduling-lifecycle.md) |
| 4 | Debug CrashLoopBackOff / pending pod | ✅ [CrashLoop](../08-kubernetes/19-crashloopbackoff.md) · ⭐ [Pending/ImagePull/node-down](../08-kubernetes/43-troubleshooting-pending-imagepull-node-down.md) |
| 5 | `terraform apply` fails halfway | ⭐ [Apply fails halfway](../06-terraform/16-apply-fails-halfway.md) |
| 6 | Secrets in Kubernetes / Terraform | ✅ [K8s secret types](../08-kubernetes/35-types-of-secrets.md) · [Secrets Mgr vs Param Store](../10-aws/22-secrets-manager-vs-parameter-store.md) · [Vault](../10-aws/119-vault-secrets-management.md) |
| 7 | Terraform state locking (why it matters) | ✅ [Concurrent state update / locking](../06-terraform/06-concurrent-statefile-update.md) |
| 8 | Logging & monitoring for microservices | ✅ [Observability tools](../09-observability/07-observability-tools.md) · [Trace across microservices](../09-observability/09-trace-request-across-microservices.md) · [Multi-cluster](../09-observability/12-multi-cluster-central-observability.md) |
| 9 | Reduce AWS cost without hurting performance | ✅ [Cost optimization](../10-aws/10-cost-optimization-activity.md) · [Core services & cost](../10-aws/108-core-services-secrets-and-cost.md) |
| 10 | Secure a CI/CD pipeline | ✅ [CI/CD deep dive (per-hop authN/authZ)](../05-cicd/17-cicd-from-scratch-gitops-deep-dive.md) · [AuthN vs AuthZ](../10-aws/29-authentication-vs-authorization.md) |
| 11 | GitOps vs traditional CI/CD | ⚠️ inline below (+ [Argo CD/GitOps](../05-cicd/16-argocd-gitops-and-image-updates.md)) |
| 12 | Recover from a failed production deployment | ✅ [Rollback strategy](../08-kubernetes/14-rollback-strategy.md) · [Incident & P1](../09-observability/14-incident-management-and-p1.md) · rollback = revert Git in [16](../05-cicd/16-argocd-gitops-and-image-updates.md) |
| 13 | Service mesh (Istio/Linkerd) vs ingress controller | ⭐ [Service mesh vs ingress](../08-kubernetes/45-service-mesh-vs-ingress.md) |
| 14 | How containers communicate inside K8s | ✅ [Pod/service communication](../08-kubernetes/04-pod-ip-communication.md) · [CoreDNS deep dive](../08-kubernetes/38-coredns-deep-dive-and-troubleshooting.md) |
| 15 | ReplicaSet vs Deployment vs StatefulSet | ⚠️ inline below (+ [Deployment/StatefulSet/DaemonSet](../08-kubernetes/42-deployment-statefulset-daemonset.md)) |
| 16 | SQS vs Kafka (when each) | ✅ [MSK — Kafka vs SQS/SNS/Kinesis](../10-aws/114-msk-managed-kafka.md) |
| 17 | Horizontal vs vertical scaling (real infra) | ✅ [K8s scaling](../08-kubernetes/34-horizontal-vs-vertical-scaling.md) · [ASG & scaling](../10-aws/115-asg-and-scaling.md) |
| 18 | Set up observability (logs/metrics/traces) | ✅ [Logs/metrics/traces](../09-observability/05-logs-metrics-traces.md) · [Tools](../09-observability/07-observability-tools.md) |
| 19 | Mutable vs immutable infrastructure | ⭐ [Mutable vs immutable](../10-aws/31-mutable-vs-immutable-infrastructure.md) |
| 20 | Service down at 2 AM — how you troubleshoot | ✅ [Incident & P1](../09-observability/14-incident-management-and-p1.md) · [Debug-order playbook](../docusign-debug-order-playbook.md) |

---

## ⚠️ Inline answers for the partials

**Q11 — GitOps vs traditional CI/CD (push vs pull)**
- **Traditional CI/CD (push):** the pipeline (Jenkins/Actions) has cluster credentials and **pushes** changes — `kubectl apply` / `helm upgrade` — *to* the cluster at the end of the run. The pipeline is the actor; there's no continuous guarantee the cluster still matches intent.
- **GitOps (pull):** **Git is the single source of truth**, and an in-cluster agent (**Argo CD/Flux**) **pulls** and **continuously reconciles** the cluster to match Git — with **drift detection** and **self-heal**. The pipeline only updates Git; it never touches the cluster directly.
- **Why GitOps wins:** cluster creds stay **inside** the cluster (smaller attack surface), every change is an auditable **Git commit**, **rollback = `git revert`**, and drift is auto-corrected. See [Argo CD/GitOps](../05-cicd/16-argocd-gitops-and-image-updates.md) and the [CI/CD deep dive](../05-cicd/17-cicd-from-scratch-gitops-deep-dive.md).
- **One-liner:** *"Traditional CI/CD **pushes** to the cluster; GitOps makes the cluster **pull** from Git and continuously reconcile — Git is the source of truth, so it's auditable, self-healing, and rollback is a revert."*

**Q15 — ReplicaSet vs Deployment vs StatefulSet**
- **ReplicaSet:** the low-level controller that keeps **N identical pod replicas** running. You rarely create it directly.
- **Deployment:** manages ReplicaSets **for you** and adds **rolling updates + rollback + versioning** (each update = a new ReplicaSet it scales up/down). This is what you actually use for stateless apps.
- **StatefulSet:** for **stateful** workloads needing **stable identity + per-pod storage + ordered rollout** (see [42](../08-kubernetes/42-deployment-statefulset-daemonset.md)).
- **One-liner:** *"A Deployment manages ReplicaSets to give you rolling updates and rollback; a ReplicaSet just keeps N identical pods alive; a StatefulSet is for stateful pods with stable identity and storage."*

---

## How to use this bank
1. Cover the right column; answer each in 60–90 seconds out loud.
2. Stumble → open the linked note, re-learn the **mechanism**, retry tomorrow.
3. Prioritize the ⭐ (were gaps until now) and anything Kubernetes — that's the depth interviewers weight most.

> Other banks & transcripts: [Common AWS DevOps/SRE bank](./05-common-aws-devops-sre-questions.md) · [Nielsen](./01-nielsen.md) · [Amagi](./02-amagi.md) · [Jio](./03-jio-cloud-security.md) · [MoveInSync](./04-moveinsync.md)
