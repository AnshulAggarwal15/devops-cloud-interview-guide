## How a Pod Gets Scheduled — the `kubectl apply` → Running Pod Lifecycle

### Question  
Walk me through what happens in the **control plane** when you create a Pod (or `kubectl apply` a Deployment) — how does it end up **scheduled and running** on a node?

### Short explanation of the question  
This is the classic "do you understand Kubernetes internals?" question. They want the **end-to-end flow**: request hits the **API server** (authN/authZ/admission) → persisted in **etcd** → **controllers** create the Pod → the **scheduler** picks a node (filter + score + **bind**) → the **kubelet** on that node starts the container via the **CRI**, wires networking via the **CNI**, and reports **Ready**. Naming each component and what it does is the whole game.

---

### Answer  
1. **API server** receives the request, runs **authentication → authorization (RBAC) → admission control**, validates it, and **writes the object to etcd**.
2. **Controllers** (via the controller-manager) react: a Deployment → ReplicaSet → **Pod objects** (still `Pending`, `nodeName` empty).
3. The **scheduler** watches for unscheduled Pods, runs **Filtering (predicates)** then **Scoring (priorities)** to pick the best node, and **binds** the Pod to it (writes `nodeName` back via the API server).
4. The **kubelet** on that node sees the Pod assigned to it, pulls the image and starts containers via the **container runtime (CRI/containerd)**, sets up networking via the **CNI**, mounts volumes via the **CSI**.
5. **Probes** pass → the Pod becomes **Ready**; **kube-proxy** and **EndpointSlices** add it to its Service so it receives traffic.

The theme: **everything goes through the API server and is stored in etcd; components watch and react.**

---

### Detailed explanation of the answer for readers’ understanding

### 🔄 The full flow
```text
kubectl apply
   │  (1) HTTPS request
   ▼
API SERVER  ── authN (who?) → authZ/RBAC (allowed?) → admission (mutate/validate) → validate schema
   │  writes desired state
   ▼
etcd  (the single source of truth; only the API server talks to it)
   │
   ▼
CONTROLLER MANAGER  ── Deployment ctrl → ReplicaSet → creates Pod objects (Pending, no node)
   │
   ▼
SCHEDULER  ── watches Pending pods → FILTER feasible nodes → SCORE them → BIND to best node
   │  (writes pod.spec.nodeName back through the API server → etcd)
   ▼
KUBELET (on the chosen node)  ── sees "pod assigned to me"
   │  → pulls image → CRI (containerd) starts container
   │  → CNI plugin gives the pod an IP / wires networking
   │  → CSI mounts any volumes
   │  → runs liveness/readiness probes
   ▼
Pod Running & Ready → kube-proxy / EndpointSlice adds it to its Service → serves traffic
```

### 🧩 What each piece does
| Component | Role in this flow |
|---|---|
| **API server** | Front door; **authN → authZ (RBAC) → admission**; the *only* component that reads/writes **etcd** |
| **etcd** | Consistent key-value store holding **all cluster state** (desired + status) |
| **Controller manager** | Reconciliation loops; turns a Deployment into a ReplicaSet into **Pod objects** |
| **Scheduler** | Assigns Pending pods to nodes via **filter → score → bind** |
| **kubelet** | Node agent; makes the node's reality match the pods assigned to it |
| **container runtime (CRI)** | containerd/CRI-O actually **runs** the container |
| **CNI / CSI** | Pod **networking** (IP) / **storage** (volume mounts) |
| **kube-proxy** | Programs Service ClusterIP → pod routing (iptables/IPVS) |

### 🎯 The scheduler in detail (the part they drill)
The scheduler decides *where* a Pending pod runs, in two phases:
1. **Filtering (predicates):** eliminate nodes that **can't** host the pod — insufficient CPU/memory (vs **requests**), **taints** without matching tolerations, **nodeSelector/affinity** mismatch, no matching volume/topology. Result: a list of *feasible* nodes.
2. **Scoring (priorities):** rank the feasible nodes — spread across nodes/zones, least-loaded, affinity preferences, image locality. Highest score wins.
3. **Binding:** the scheduler writes `nodeName` onto the pod (through the API server). It **doesn't** start the container — it only *decides*.

> Key insight: the scheduler only **assigns**; the **kubelet** on the chosen node does the actual running. If no node passes filtering, the pod stays **Pending** (see the Pending troubleshooting note).

### 🔁 "Watch and reconcile" is the mental model
Nothing pushes commands around. Each component **watches the API server** for state it cares about and **reconciles** reality toward desired state. `kubectl apply` just records desired state in etcd; the controllers, scheduler, and kubelet make it real asynchronously. This is why Kubernetes is **declarative and self-healing** — kill a pod and the loop recreates it.

### ⚠️ Things they probe
- **Only the API server talks to etcd** — components don't hit etcd directly.
- The **scheduler binds, the kubelet runs** — don't conflate them.
- Requests (not limits) drive **scheduling** decisions; a pod with no fitting node stays **Pending**.
- Order at the API server: **authN → authZ → admission → persist**. (Ties to the [authN/authZ framework](../10-aws/29-authentication-vs-authorization.md).)

---

### 🧪 Real-World Example

> "When I `kubectl apply` a Deployment, the API server authenticates me, checks RBAC, runs admission, and stores it in etcd. The Deployment controller creates a ReplicaSet, which creates Pods in `Pending`. The scheduler filters nodes by requests/taints/affinity, scores the survivors, and binds each pod to a node. That node's kubelet pulls the image, containerd starts it, the CNI assigns a pod IP, probes pass, and kube-proxy/EndpointSlices route Service traffic to it."

---

### Key takeaway

> "**API server (authN→authZ→admission) → etcd → controllers create Pods → scheduler filters+scores+binds a node → kubelet + CRI run the container, CNI/CSI wire net/storage → probes pass → kube-proxy adds it to the Service.** Only the API server touches etcd, every component **watches and reconciles**, and the **scheduler decides while the kubelet runs**."

### Related
- [Cluster architecture](./01-cluster-architecture.md) · [Components interaction](./02-components-interaction.md)
- [Can the master schedule pods? (taints)](./33-master-for-scheduling-pods.md) · [Pending/ImagePull/node-down troubleshooting](./43-troubleshooting-pending-imagepull-node-down.md)
- [Authentication vs Authorization framework](../10-aws/29-authentication-vs-authorization.md)
