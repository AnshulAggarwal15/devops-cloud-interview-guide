## What are Requests and Limits in Kubernetes?

### Question  
What are resource `requests` and `limits` in Kubernetes, and what happens when a pod exceeds them?

### Short explanation of the question  
This tests your understanding of how Kubernetes **reserves and caps** CPU and memory for containers, how requests drive **scheduling**, and the very different behavior of exceeding a **CPU** limit vs a **memory** limit.

---

### Answer  
- **Requests** are the resources a container is **guaranteed and reserved**; the **scheduler** uses them to decide which node a pod fits on.
- **Limits** are the **maximum** a container may use.

If a container exceeds its **CPU** limit it gets **throttled** (slowed down); if it exceeds its **memory** limit it gets **OOMKilled** (terminated).

---

### Detailed explanation of the answer for readers’ understanding

### 🧱 Requests vs Limits

| | Request | Limit |
|---|---|---|
| Meaning | Guaranteed/reserved amount | Hard maximum |
| Used for | **Scheduling** (finding a node) | **Capping** runtime usage |
| If exceeded (CPU) | — | **Throttled** |
| If exceeded (memory) | — | **OOMKilled** |

```yaml
resources:
  requests:      # reserved + used for scheduling
    cpu: "250m"
    memory: "128Mi"
  limits:        # hard cap
    cpu: "500m"
    memory: "256Mi"
```

### 📉 Why requests matter for scheduling
The scheduler places a pod only on a node with enough **unreserved** capacity to satisfy its **requests** — not its actual live usage. Set requests too high and pods stay `Pending`; too low and nodes get overcommitted.

### ⚠️ CPU vs Memory behavior (the key detail)
- **CPU is compressible** → exceeding the limit just **throttles** the container. It slows down but keeps running.
- **Memory is not compressible** → exceeding the limit triggers an **OOMKill**, and you'll see the container restart with `OOMKilled`.

### 🔗 Relation to QoS
Requests/limits also determine the pod's **QoS class** (Guaranteed / Burstable / BestEffort), which sets eviction priority.

### 🔎 Debugging CPU throttling
CPU throttling is **silent** — pods aren't killed, they just slow down. To spot it:
- Compare the container metrics **`container_cpu_cfs_throttled_periods_total`** vs `container_cpu_cfs_periods_total` — a high ratio means throttling (Grafana has throttling panels for this).
- Symptom: high latency/timeouts while **CPU sits right at the limit** and memory looks fine.
- Fix: **raise the CPU limit** (or drop the limit for latency-sensitive workloads and rely on requests), right-size from observed usage, or scale out.

---

### 🧪 Real-World Example

> “A service kept restarting with `OOMKilled`. Its memory **limit** was 256Mi but under load it needed ~400Mi. We raised the limit and set a matching request so the scheduler reserved enough. The restarts stopped, and the pod moved from BestEffort to Burstable.”

---

### Key takeaway

> “**Requests** = reserved amount used for **scheduling**; **limits** = hard cap. Exceeding a **CPU** limit **throttles**; exceeding a **memory** limit gets you **OOMKilled**. Setting them correctly also controls your pod's **QoS**.”
