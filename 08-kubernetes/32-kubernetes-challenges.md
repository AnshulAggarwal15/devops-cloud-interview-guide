## Explain 3 Challenges You Faced While Working on Kubernetes

### Question  
Explain 3 challenges that you faced while working on Kubernetes and how you solved them.

### Short explanation of the question  
This is an **experience** question. The interviewer wants real, specific problems — not textbook definitions — told as **symptom → investigation → root cause → fix**. Pick challenges that show depth across scheduling, networking, and resources.

---

### Answer  
I usually talk about three real ones: **pods stuck in Pending due to resources/affinity**, **intermittent DNS resolution failures (CoreDNS)**, and **OOMKilled pods from missing memory limits**. For each I explain how I found the root cause and the permanent fix.

---

### Detailed explanation of the answer for readers’ understanding

### 🧩 Challenge 1 — Pods stuck in `Pending`
- **Symptom:** New pods never scheduled.
- **Investigation:** `kubectl describe pod` showed `Insufficient cpu/memory` (or a node-affinity/taint mismatch).
- **Root cause:** Requests were too high for available node capacity; sometimes a taint with no matching toleration.
- **Fix:** Right-sized requests, added nodes via **Cluster Autoscaler**, and corrected tolerations/affinity.

### 🌐 Challenge 2 — Intermittent DNS failures
- **Symptom:** Services occasionally failed to resolve `*.svc.cluster.local`.
- **Investigation:** Errors pointed to **CoreDNS**; CoreDNS pods were throttled/restarting.
- **Root cause:** CoreDNS was under-resourced and running too few replicas for the cluster's query load.
- **Fix:** Scaled CoreDNS replicas, set proper requests/limits, and tuned `ndots`/caching. Resolution stabilized.

### 💥 Challenge 3 — Pods `OOMKilled`
- **Symptom:** A service kept restarting; `kubectl describe` showed `OOMKilled`.
- **Investigation:** Memory usage crossed the container's memory **limit** under load.
- **Root cause:** Limit set too low (or missing), making the pod BestEffort and easy to kill.
- **Fix:** Set correct memory **requests/limits** (moving it to Burstable/Guaranteed) and added a memory alert.

---

### 📌 How to structure the answer

```text
symptom → how I investigated (kubectl describe / logs / events)
→ root cause → permanent fix → prevention
```

---

### 🧪 Real-World Example

> “The DNS one was the trickiest — failures were intermittent so it looked like an app bug. Events on the CoreDNS pods revealed CPU throttling. Scaling CoreDNS and giving it proper resources fixed a problem three teams had been chasing.”

---

### Key takeaway

> “Pick **three concrete incidents** (scheduling `Pending`, CoreDNS/DNS, and `OOMKilled`) and tell each as **symptom → investigation → root cause → fix → prevention**. Specifics and evidence (`kubectl describe`, events, logs) are what make this answer land.”
