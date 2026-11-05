## Horizontal vs Vertical Scaling in Kubernetes (and in General)

### Question  
Explain horizontal vs vertical scaling in Kubernetes, or in general.

### Short explanation of the question  
This checks the core scaling concept — **scale out (more instances)** vs **scale up (bigger instances)** — and how Kubernetes implements each with the **HPA**, **VPA**, and **Cluster Autoscaler**.

---

### Answer  
- **Horizontal scaling (scale out)** = add **more** instances/pods to share the load.
- **Vertical scaling (scale up)** = give an existing instance/pod **more resources** (CPU/memory).

In Kubernetes, horizontal scaling is done by the **Horizontal Pod Autoscaler (HPA)**, vertical scaling by the **Vertical Pod Autoscaler (VPA)**, and node capacity by the **Cluster Autoscaler**.

---

### Detailed explanation of the answer for readers’ understanding

### 🔀 Horizontal vs Vertical

| | Horizontal (scale out) | Vertical (scale up) |
|---|---|---|
| What changes | **Number** of pods/nodes | **Size** (CPU/memory) of a pod/node |
| K8s tool | **HPA** | **VPA** |
| Downtime | None (add replicas) | Often needs a **pod restart** |
| Limit | Very high (add many replicas) | Bounded by node size |
| Best for | Stateless apps, spiky traffic | Apps that can't be replicated easily |

### 📈 Horizontal Pod Autoscaler (HPA)
Adds/removes **pod replicas** based on metrics like CPU/memory or custom metrics.
```bash
kubectl autoscale deployment web --cpu-percent=60 --min=2 --max=10
```

### 📊 Vertical Pod Autoscaler (VPA)
Adjusts a pod's **requests/limits** to right-size it. Applying new resources usually means the pod is **recreated**.

### 🖥️ Cluster Autoscaler
Horizontal scaling at the **node** level — adds nodes when pods can't be scheduled, removes them when idle. Often paired with HPA (more pods → need more nodes).

### 🧭 Which to use?
- Prefer **horizontal** for stateless web/API workloads — no downtime, near-unlimited headroom.
- Use **vertical** for workloads that are hard to replicate (some stateful/single-instance apps).
- Combine **HPA + Cluster Autoscaler** for elastic, cost-efficient scaling.

---

### 🧪 Real-World Example

> “Our API saw traffic spikes at peak hours. We set an **HPA** (2–10 replicas at 60% CPU) so pods scaled out automatically, and the **Cluster Autoscaler** added nodes when needed. No manual intervention, and it scaled back down after the peak to save cost.”

---

### Key takeaway

> “**Horizontal = more pods (HPA), Vertical = bigger pods (VPA)**, with the **Cluster Autoscaler** adding nodes. For stateless apps, prefer horizontal scaling; combine HPA with the Cluster Autoscaler for elastic, cost-efficient scaling.”

---

### Related
- [Advanced autoscaling — custom metrics, KEDA, multiple triggers & stabilization](./37-advanced-autoscaling-custom-metrics-keda.md) — scaling on Prometheus/event metrics, KEDA vs HPA conflicts, trigger precedence, and stopping startup-lag overshoot.
