## Cordon, Drain, and Taint — Moving Workloads Off a Node

### Question  
What's the difference between **taint** and **cordon** (and **drain**)? In production, how would you move workloads from an **unhealthy node to a healthy one**?

### Short explanation of the question  
These all influence *where pods run*, but at different moments. **Cordon** = mark a node unschedulable (no *new* pods). **Drain** = cordon **plus evict** the existing pods so they reschedule elsewhere. **Taint** = a repelling property on a node that pods must **tolerate** to land there. Moving workloads off a node for maintenance = **cordon + drain**.

---

### Answer  
- **Cordon** (`kubectl cordon <node>`): marks the node **unschedulable** — no **new** pods land, but **existing pods keep running**.
- **Drain** (`kubectl drain <node>`): cordons **and evicts** the running pods (respecting PodDisruptionBudgets) so they're rescheduled onto other nodes — used before maintenance/decommission.
- **Taint** (`kubectl taint`): a node property that **repels** pods unless they have a matching **toleration** — used for **dedicated/specialized nodes** (GPU, isolation), applied proactively.
- **Move workloads unhealthy → healthy node:** **`cordon`** the bad node (stop new pods), then **`drain`** it (evict existing pods → they reschedule on healthy nodes), fix/replace the node, then **`uncordon`** to allow scheduling again.

---

### Detailed explanation of the answer for readers’ understanding

### 🔀 The three, compared
| | Cordon | Drain | Taint |
|---|---|---|---|
| What it does | Mark node **unschedulable** | Cordon **+ evict** existing pods | **Repel** pods lacking a toleration |
| Existing pods | **Stay running** | **Evicted/rescheduled** | Stay (unless `NoExecute`) |
| New pods | Blocked | Blocked | Blocked unless they **tolerate** |
| Typical use | Pause scheduling on a node | **Maintenance/decommission** | **Dedicated nodes** (GPU/isolation) |
| Command | `kubectl cordon` | `kubectl drain` | `kubectl taint` |

**Cordon vs taint in one line:** *cordon* is a blunt on/off "no new pods here" toggle for a specific node during ops; *taint* is a selective, policy-style repel that only pods with the right *toleration* can bypass. (See [taints & tolerations](./18-taints-tolerations.md).)

### 🚚 Moving workloads off a node (the production procedure)
```bash
# 1. Stop new pods landing on the bad node
kubectl cordon node-1

# 2. Evict running pods → they reschedule onto healthy nodes
kubectl drain node-1 --ignore-daemonsets --delete-emptydir-data
#   --ignore-daemonsets: DaemonSet pods can't move (one per node) — expected
#   --delete-emptydir-data: allow eviction of pods using emptyDir (data is lost)
#   drain honors PodDisruptionBudgets, so it evicts gradually to keep min replicas up

# 3. Do the maintenance / replace the node (or let the ASG/Karpenter replace it)

# 4. Allow scheduling again (if the node returns)
kubectl uncordon node-1
```
- Pods only reschedule cleanly if there's **spare capacity** and the app runs **multiple replicas** (single-replica → brief downtime). A **PodDisruptionBudget** ensures drain doesn't take too many replicas down at once.
- On EKS with **managed node groups / Karpenter**, the usual flow is drain → terminate the instance → the autoscaler launches a fresh node. Immutable-infra style: **replace, don't repair**.

### 🩺 If the node is already unhealthy (NotReady)
- If the kubelet is dead/partitioned, the node goes **NotReady** and the controller **evicts** its pods automatically after the timeout — you may just **cordon** to stop churn and **replace** the instance. (See [node-down troubleshooting](./43-troubleshooting-pending-imagepull-node-down.md).)
- `drain` may hang if the node is unreachable; then you delete the node object and let the ASG replace it.

### ⚠️ Things they probe
- **Cordon = no new pods (existing stay); Drain = cordon + evict; Taint = repel unless tolerated.**
- `drain` needs **`--ignore-daemonsets`** (DaemonSet pods can't be moved) and honors **PDBs**.
- Safe moves require **multi-replica + spare capacity + PDB**; otherwise you get downtime.
- `uncordon` to re-enable; on cloud, often you just **replace** the node.

---

### 🧪 Real-World Example

> "A node showed disk errors. I **cordoned** it so nothing new scheduled there, then **drained** it with `--ignore-daemonsets`; because our apps ran 3 replicas across AZs with PDBs, pods moved to healthy nodes with zero downtime. We terminated the instance and the managed node group launched a fresh one — no in-place repair."

---

### Key takeaway

> "**Cordon** = mark unschedulable (existing pods stay); **drain** = cordon **+ evict** (for maintenance, honors PDBs, needs `--ignore-daemonsets`); **taint** = repel pods unless they **tolerate** it (dedicated nodes). To move workloads off a bad node: **cordon → drain → replace/fix → uncordon**, and rely on **multi-replica + spare capacity + PDB** for zero downtime."

### Related
- [Taints & tolerations](./18-taints-tolerations.md) · [Pending/ImagePull/node-down troubleshooting](./43-troubleshooting-pending-imagepull-node-down.md)
- [Pod scheduling lifecycle](./44-pod-scheduling-lifecycle.md) · [Mutable vs immutable infra](../10-aws/31-mutable-vs-immutable-infrastructure.md)
