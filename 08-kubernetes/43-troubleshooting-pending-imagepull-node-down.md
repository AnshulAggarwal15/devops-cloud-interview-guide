## Troubleshooting: Pod Pending, ImagePullBackOff, and Node Down

### Question  
How do you troubleshoot a pod stuck in **Pending**, a pod in **ImagePullBackOff**, and what happens (and what do you do) when a **worker node goes down**?

### Short explanation of the question  
These are the three most common "pod won't run" scenarios after CrashLoopBackOff. Each has a **distinct root-cause family**: **Pending = can't be scheduled**, **ImagePullBackOff = can't fetch the image**, **node down = pods need to be rescheduled elsewhere**. The method is the same every time: **`kubectl describe` → read the events → follow the specific cause.**

---

### Answer  
- **Pending** = the scheduler can't place the pod: **no node has room** (CPU/mem requests), a **taint/affinity/selector** excludes nodes, or a **PVC/volume** can't bind. Check `describe pod` events.
- **ImagePullBackOff** = the kubelet can't pull the image: **wrong name/tag**, **no registry auth** (ECR/private), or **network/rate-limit**. Check `describe pod` events.
- **Node down** = the node goes **NotReady**; after a timeout the controller **evicts** its pods and reschedules them on healthy nodes (if capacity + no constraints). You investigate the node (kubelet/network/resources) and ensure capacity + PDBs let workloads move.

Always start with **`kubectl describe pod`** (Events) and **`kubectl get events`** — the reason is almost always printed there.

---

### Detailed explanation of the answer for readers’ understanding

### 🟡 Pod stuck in Pending (can't be scheduled)
`kubectl describe pod <p>` → read the Events (e.g. `0/5 nodes are available: insufficient cpu`, `node(s) had taint …`, `pod has unbound PVC`).
| Cause | Signal | Fix |
|---|---|---|
| **No capacity** | "insufficient cpu/memory" | Lower requests, add nodes (Cluster Autoscaler/Karpenter), or bigger nodes |
| **Taints** | "had taint {…} that the pod didn't tolerate" | Add a **toleration** or use a different node pool |
| **Affinity / nodeSelector** | "didn't match node affinity/selector" | Fix the label/affinity rule |
| **Unbound PVC** | "pod has unbound immediate PersistentVolumeClaims" | Fix the StorageClass / provisioner; ensure a PV can bind |
| **Topology / anti-affinity** | "didn't match pod anti-affinity" | Relax spread rules or add nodes in other zones |
```bash
kubectl describe pod <p>            # Events tell you which of the above
kubectl get events --sort-by=.lastTimestamp
kubectl top nodes                   # capacity pressure?
```

### 🔴 ImagePullBackOff / ErrImagePull (can't fetch image)
`kubectl describe pod <p>` → the Events show the pull error.
| Cause | Signal | Fix |
|---|---|---|
| **Wrong image/tag** | "manifest unknown / not found" | Fix repo/tag; ensure CI actually pushed it |
| **No registry auth** | "pull access denied / unauthorized / no basic auth" | Add **`imagePullSecrets`**, or for ECR use the **node role / IRSA** with `ecr:GetDownloadUrlForLayer`, `ecr:BatchGetImage` |
| **Private registry / network** | timeouts | Egress/NAT to the registry, VPC endpoint for ECR |
| **Docker Hub rate limit** | "toomanyrequests" | Authenticate, or mirror/cache the image |
```bash
kubectl describe pod <p>            # see the exact pull error
kubectl get sa <sa> -o yaml         # imagePullSecrets present?
```
> `BackOff` just means the kubelet is **retrying with exponential backoff** after repeated pull failures — fix the underlying cause and it self-resolves.

### 🟠 Worker node goes down
**What happens:** the node stops heartbeating → marked **`NotReady`**. After `--node-monitor-grace-period` (~40s) it's tainted `node.kubernetes.io/not-ready`; after the **eviction timeout** (~5 min default) the pods are **evicted and rescheduled** onto healthy nodes — *if* there's capacity and no scheduling constraints block them. Pods on a dead node may show `Terminating`/`Unknown` until then.
**Investigate the node:**
```bash
kubectl get nodes                   # which node is NotReady?
kubectl describe node <node>        # conditions: MemoryPressure/DiskPressure/PIDPressure? taints?
```
Common root causes: **kubelet crashed/stopped**, **network partition** (node can't reach API server), **resource exhaustion** (disk/memory pressure), or the **EC2 instance failed** (in EKS the ASG/managed node group replaces it).
**Make workloads survive it:**
- Run **multiple replicas across nodes/AZs** (topology spread / anti-affinity).
- Set a **PodDisruptionBudget** so voluntary disruptions keep a minimum available.
- Ensure **spare capacity / autoscaler** so evicted pods have somewhere to land.
- StatefulSet pods reattach to **their PVC** on the new node (EBS is AZ-bound — reschedule must stay in-AZ).

### 🧭 The universal method (say this)
> "For any 'pod not running' issue I start with **`kubectl describe pod`** and read the **Events** — Pending points me at scheduling (capacity/taints/affinity/PVC), ImagePull points me at the registry (name/auth/network), and a NotReady node points me at the kubelet/network/resources on that node. I don't guess; the events name the cause."

### ⚠️ Things they probe
- **Pending ≠ CrashLoop**: Pending never started (scheduling); CrashLoop started and keeps dying (app/config). Don't confuse them.
- ImagePullBackOff on EKS is very often **missing ECR permissions** on the node role / IRSA.
- Node-down recovery depends on **capacity + PDB + spread** — a single-replica app on a dead node = downtime until rescheduled.

---

### 🧪 Real-World Example

> "A deploy sat **Pending** — `describe` showed 'insufficient cpu'; Karpenter hadn't scaled yet, so we right-sized requests and it scheduled. Another time pods hit **ImagePullBackOff**: `describe` said 'unauthorized' — the node role was missing ECR read perms after a role change. When a node went **NotReady** (kubelet OOM), pods rescheduled automatically because we ran 3 replicas across AZs with a PDB, so users never noticed."

---

### Key takeaway

> "**`describe pod` + Events first, every time.** **Pending = scheduling** (capacity/taints/affinity/PVC), **ImagePullBackOff = image fetch** (name/tag/registry auth/network — often ECR perms on EKS), **node down → NotReady → pods evicted & rescheduled** after the timeout, so survive it with **multi-replica across AZs + PDB + spare capacity**."

### Related
- [CrashLoopBackOff](./19-crashloopbackoff.md) · [Liveness vs readiness](./20-liveness-vs-readiness.md) · [Requests & limits](./31-requests-and-limits.md)
- [Taints & tolerations](./18-taints-tolerations.md) · [Node affinity](./27-node-affinity.md)
- [Pod IAM / IMDS troubleshooting (ECR/creds)](../10-aws/27-pod-node-role-imds-troubleshooting.md)
