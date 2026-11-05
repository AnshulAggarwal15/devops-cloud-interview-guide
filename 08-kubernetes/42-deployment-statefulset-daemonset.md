## Deployment vs StatefulSet vs DaemonSet

### Question  
What's the difference between a **Deployment**, a **StatefulSet**, and a **DaemonSet**, and when do you use each?

### Short explanation of the question  
All three manage pods, but for **different workload shapes**: interchangeable stateless replicas (Deployment), pods with **stable identity + storage** (StatefulSet), and **one pod per node** (DaemonSet). Interviewers want you to pick the right one for a given app and justify it.

---

### Answer  
- **Deployment** → **stateless** apps: N identical, interchangeable replicas; scale/roll/rollback freely. (Web/API tiers.)
- **StatefulSet** → **stateful** apps needing **stable network identity, ordered startup, and per-pod persistent storage**. (Databases, Kafka, ZooKeeper.)
- **DaemonSet** → **one pod per node** (or per matching node): node-level agents. (Log shippers, CNI, node exporters.)

---

### Detailed explanation of the answer for readers’ understanding

### 🔀 Side by side
| | Deployment | StatefulSet | DaemonSet |
|---|---|---|---|
| Pod identity | Random names, interchangeable | **Stable, ordinal** (`app-0`, `app-1`) | One per node |
| Storage | Shared/none (ephemeral) | **Per-pod PVC** (`volumeClaimTemplates`), stays with the ordinal | Usually hostPath/none |
| Startup/scaling order | Any order, parallel | **Ordered** (0→1→2), one at a time | Follows nodes |
| Network | Via Service (any pod) | **Stable DNS per pod** via headless Service (`app-0.svc…`) | Node-local |
| Scales by | replica count | replica count (ordered) | **node count** (automatic) |
| Use for | Stateless web/API/workers | Databases, queues, clustered stateful | Node agents (logs/metrics/CNI) |

### 🟦 Deployment — the default
Manages a ReplicaSet of **identical, disposable** pods. Any replica serves any request; scaling and **rolling updates/rollbacks** are trivial. This is what you use for a typical stateless service.

### 🟩 StatefulSet — identity + storage
Use when each pod must be **distinguishable and keep its own data**:
- **Stable ordinal identity**: `mysql-0`, `mysql-1` — names survive rescheduling.
- **Stable per-pod DNS** via a **headless Service**: `mysql-0.mysql.ns.svc.cluster.local` — critical for clustering (primary/replica, peers finding each other).
- **Per-pod persistent volume** via `volumeClaimTemplates`: `mysql-0` always reattaches to *its* PVC, even after a restart on another node.
- **Ordered, graceful** rollout/scale (0 up first, deleted in reverse) — matters for quorum-based systems.

### 🟨 DaemonSet — one per node
Ensures a copy runs on **every node** (or every node matching a selector/taint). New node joins → the pod is scheduled automatically. Classic uses: **Fluent Bit/Fluentd** (logs), **node-exporter** (metrics), **CNI agents**, **NodeLocal DNSCache**, security agents.

### 🧭 How to choose (say this)
- Can any replica handle any request and hold no local state? → **Deployment**.
- Does each instance need a **stable name, stable network ID, or its own durable disk**, or must start in order? → **StatefulSet**.
- Does it need to run **on every node** as an agent? → **DaemonSet**.

### ⚠️ Things they probe
- "StatefulSet gives HA" → no; it gives **identity + storage + ordering**. HA still comes from replicas across nodes/AZs.
- StatefulSet PVCs are **not** deleted automatically when you scale down (data safety) — clean up deliberately.
- Prefer **managed data stores (RDS/Aurora/MSK)** over in-cluster StatefulSets when you can — running stateful systems on K8s is operationally heavy.
- (Jobs/CronJobs exist too, for batch/one-shot workloads.)

---

### 🧪 Real-World Example

> "Our API tier was a **Deployment** (10 stateless replicas behind a Service, rolling updates). Our self-hosted **Kafka** ran as a **StatefulSet** so each broker kept a stable ID and its own EBS volume via `volumeClaimTemplates`. **Fluent Bit** ran as a **DaemonSet** so every node shipped its container logs. Where we could, we moved stateful stores to **RDS/MSK** to avoid operating StatefulSets ourselves."

---

### Key takeaway

> "**Deployment = interchangeable stateless replicas; StatefulSet = stable identity + per-pod storage + ordered rollout (databases/queues); DaemonSet = one pod per node (agents).** Choose by whether the workload is stateless, needs durable per-instance identity/storage, or must run on every node — and prefer managed data stores over in-cluster StatefulSets when possible."

### Related
- [Deployment strategies](./16-all-deployment-strategies.md) · [Types of services (headless)](./05-types-of-services.md) · [Headless service](./10-headless-service.md)
- [App → Kubernetes production readiness](./41-app-to-kubernetes-production-readiness.md)
