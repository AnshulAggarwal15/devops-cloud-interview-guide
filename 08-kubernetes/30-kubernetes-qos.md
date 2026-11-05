## What is Kubernetes QoS (Quality of Service)?

### Question  
What is Quality of Service (QoS) in Kubernetes and how is it decided?

### Short explanation of the question  
This checks whether you understand how Kubernetes **classifies pods** based on their resource `requests` and `limits`, and how that classification decides **which pods get evicted first** when a node runs out of resources.

---

### Answer  
**QoS** is a classification Kubernetes assigns to each pod — **Guaranteed**, **Burstable**, or **BestEffort** — based on how its CPU/memory **requests and limits** are set. It determines the **eviction priority** under node resource pressure: **BestEffort pods are killed first, Guaranteed pods last**.

---

### Detailed explanation of the answer for readers’ understanding

Kubernetes decides QoS automatically from the pod's resource spec.

### 🧱 The three QoS classes

| QoS Class | Condition | Eviction priority |
|-----------|-----------|-------------------|
| **Guaranteed** | Every container has **requests == limits** for **both** CPU and memory | Evicted **last** |
| **Burstable** | Has at least one request/limit, but not equal for all | Evicted **after BestEffort** |
| **BestEffort** | **No** requests or limits set at all | Evicted **first** |

### ✅ Guaranteed example
```yaml
resources:
  requests:
    cpu: "500m"
    memory: "256Mi"
  limits:
    cpu: "500m"       # equal to requests
    memory: "256Mi"   # equal to requests
```

### ⚖️ Burstable example
```yaml
resources:
  requests:
    cpu: "250m"
    memory: "128Mi"
  limits:
    cpu: "500m"       # higher than request → Burstable
    memory: "256Mi"
```

### 🚫 BestEffort
No `requests` or `limits` at all → first to be evicted when the node is under memory pressure.

---

### 🧪 Real-World Example

> “Under memory pressure a node started evicting pods. Our critical payment pod survived because it was **Guaranteed** (requests == limits), while a **BestEffort** batch job with no limits got killed first. After that we always set requests/limits on critical workloads to push them into Guaranteed/Burstable.”

---

### Key takeaway

> “QoS (Guaranteed, Burstable, BestEffort) is derived from how you set **requests and limits**. It controls **eviction order** under resource pressure — set `requests == limits` for critical pods to make them **Guaranteed** and evicted last.”
