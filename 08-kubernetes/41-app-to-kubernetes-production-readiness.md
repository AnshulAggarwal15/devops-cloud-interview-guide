## Migrating a Microservice to Kubernetes — Production-Readiness Plan

### Question  
Management says: "Move this microservice onto Kubernetes. Give me a **plan**, and make sure it's **production-grade and secure**." (And the follow-up: which Kubernetes **objects/files** do you actually configure?)

### Short explanation of the question  
They want your **migration *approach and plan*** — not an application-discovery checklist. A common misfire is answering "first I'd study the traffic pattern and whether it uses RDS/Redis…"; that's discovery, and the interviewer will cut you off with "I'm asking for your **plan**." Lead with the **migration path** (containerize → non-prod → harden → controlled cutover), then layer in production concerns.

---

### Answer  
> "My plan: **containerize** the service without changing business logic, deploy it to a **non-production cluster**, then wire up **networking, config/secrets, resource limits, health probes, observability, and autoscaling**. Once it's functional and **load-tested**, I integrate it into **CI/CD** and migrate production with a **controlled rollout** — a small % of traffic first, validate metrics/errors, then ramp to 100%. If anything looks wrong, I **roll traffic back** to the old environment."

**Flow:** `Assess → Containerize → Manifests → Security → Observability → Autoscaling → CI/CD → Test → Controlled rollout → Rollback plan`

---

### Detailed explanation of the answer for readers’ understanding

### 🧭 The plan (say it as a sequence)
1. **Assess (briefly):** dependencies (RDS/Redis/queues), statefulness, traffic profile, availability target. Keep this short — it *informs* the plan, it isn't the plan.
2. **Containerize:** a multi-stage `Dockerfile`, run as **non-root**, minimal base image, config via env, **no secrets baked in**. Same artifact promoted across envs.
3. **Kubernetes manifests:** Deployment/Service/Config/Secrets (details below).
4. **Security:** RBAC, NetworkPolicies, Pod Security, **IRSA** for AWS access, TLS, image scanning.
5. **Observability:** logs, metrics, health endpoints, alerts — from day one, not after.
6. **Autoscaling:** requests/limits → HPA (→ KEDA if event-driven) → Cluster Autoscaler/Karpenter for nodes.
7. **CI/CD:** build → scan → push → **GitOps deploy (Argo CD)**.
8. **Test in lower envs:** functional + **load + failure testing**.
9. **Controlled production cutover:** canary/weighted traffic, validate, ramp; keep the old env as **rollback**.

### 📦 Which Kubernetes objects/files (the follow-up)
Don't create objects blindly — they follow the app's needs:
| Object | When / why |
|---|---|
| **Deployment** | Always — image, replicas, ports, **resource requests/limits**, **readiness/liveness probes**, **securityContext**, env |
| **Service** | Always — stable networking (`ClusterIP` internal; behind Ingress for external) |
| **ConfigMap** | Non-sensitive config |
| **Secret / ExternalSecret** | Credentials — ideally synced from **Secrets Manager/Vault** (External Secrets Operator), not raw |
| **Ingress / Gateway** | Only if external HTTP(S) — with **TLS** + DNS |
| **HPA** | If it needs to scale on load |
| **PVC** | Only if it needs persistent storage |
| **ServiceAccount + RBAC / IRSA** | If it needs K8s API or **AWS** permissions |
| **PodDisruptionBudget** | Keep min replicas available during disruptions |
| **NetworkPolicy** | Restrict who can talk to it (default-deny) |

A typical layout:
```text
k8s/
├── deployment.yaml     # image, replicas, probes, requests/limits, securityContext
├── service.yaml        # ClusterIP
├── configmap.yaml      # non-sensitive config
├── externalsecret.yaml # pulls from Secrets Manager/Vault
├── ingress.yaml        # only if external + TLS
├── hpa.yaml            # if autoscaling
└── pdb.yaml            # availability during rollouts/drains
```
*"A stateless Python API might need only Deployment + Service + ConfigMap; persistence, external access, autoscaling, or AWS access add the other objects."*

### 🔒 Production security checklist (keywords to say)
`RBAC | NetworkPolicy (default-deny) | Secrets via Vault/Secrets Manager | IRSA (no baked keys) | TLS | image scanning | non-root + securityContext | least privilege`

### 🛡️ Reliability checklist
`readiness/liveness probes | requests & limits | HPA | PodDisruptionBudget | multi-replica across AZs | rolling/canary | monitoring + alerts | tested rollback`

### 🚦 The controlled cutover (don't big-bang)
- Run new (K8s) and old side by side; shift traffic gradually (weighted DNS/ALB, service mesh, or ingress canary).
- Watch **SLOs (latency/error rate)**; ramp if healthy, **roll traffic back** instantly if not.
- Decommission the old environment only after a soak period.

### ⚠️ Things they probe
- Give the **plan/approach**, not app discovery — if you catch yourself listing "what does it depend on," pivot to "here's how I'd move it."
- **Objects depend on requirements** — don't recite every object for a simple stateless service.
- **Controlled rollout + rollback** is the senior signal; big-bang cutovers are a red flag.

---

### 🧪 Real-World Example

> "We moved a stateless Python API to EKS. Containerized as-is (non-root, multi-stage), deployed to a **staging** cluster with Deployment + Service + ConfigMap + an **ExternalSecret** from Secrets Manager, added **probes, requests/limits, HPA, a PDB**, and **NetworkPolicies**. AWS access was via **IRSA**. After load-testing, we shipped it through **Argo CD** and cut prod over with an **ingress canary** — 10% traffic, watched error rate/latency, then 100%. The old service stayed warm for a day as rollback."

---

### Key takeaway

> "Lead with the **migration path — containerize → non-prod → harden (security/observability/autoscaling) → CI/CD → load test → controlled canary cutover with rollback** — not application discovery. The **Kubernetes objects follow the app's needs** (Deployment/Service/Config always; Secrets/Ingress/HPA/PVC/RBAC as required), and production means **probes, limits, HPA, PDB, NetworkPolicy, IRSA, TLS, and scanning**."

### Related
- [CI/CD from scratch & GitOps](../05-cicd/17-cicd-from-scratch-gitops-deep-dive.md)
- [Deployment strategies](./16-all-deployment-strategies.md) · [Horizontal vs vertical scaling](./34-horizontal-vs-vertical-scaling.md)
- [Cluster hardening](./40-cluster-hardening.md) · [EKS migration (cluster-level)](../10-aws/120-eks-migration.md)
