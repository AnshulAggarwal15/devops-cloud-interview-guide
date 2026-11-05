## A Pod Crashes Randomly with OOMKilled — How Do You Identify and Fix It?

### Question  
A pod crashes randomly with `OOMKilled`. How do you identify and fix this?

### Short explanation of the question  
`OOMKilled` means the container exceeded its **memory limit** and the kernel killed it. The interviewer wants to see you **confirm** it via events/metrics, understand **why** (limit too low or a memory leak), and apply the right fix.

### Answer  
I confirm the `OOMKilled` from `kubectl describe` and memory metrics, then determine whether the **memory limit is simply too low** for normal usage or the app has a **memory leak**. Fix is either **raise requests/limits** to fit real usage or **fix the leak**; I also add a **memory alert** so it's caught before crashing.

### Detailed explanation of the answer for readers’ understanding

#### 1. 🔎 Confirm it's OOM
```bash
kubectl describe pod <pod>       # Last State: Terminated, Reason: OOMKilled
kubectl get pod <pod> -o jsonpath='{.status.containerStatuses[*].lastState}'
```
Also check **memory usage metrics** (Prometheus/`kubectl top`) leading up to the crash.

#### 2. 🧠 Figure out why — two patterns

| Pattern | Signal | Fix |
|---|---|---|
| **Limit too low** | Steady memory that normally exceeds the limit under load | Raise `requests`/`limits` to fit real usage |
| **Memory leak** | Memory **climbs steadily** until the limit, then OOM, repeat (sawtooth) | Fix the app (profiling), as a stopgap increase limit / add restarts |

#### 3. 🛠️ Apply the fix
- Set correct **requests and limits** based on observed peak usage (with headroom):
```yaml
resources:
  requests: { memory: "512Mi" }
  limits:   { memory: "1Gi" }
```
- If it's a **leak**, profile the app (heap dumps) and fix the root cause — just raising the limit only delays the crash.
- Setting `requests == limits` also raises the pod to **Guaranteed** QoS (less likely to be evicted).

#### 4. 🔔 Prevent recurrence
- Alert on **container memory usage vs limit** (e.g. > 85%).
- Alert on **pod restarts / OOMKilled events** so you catch it before users do.

---

### 🧠 Real-World Example

> A service OOMKilled a few times a day. Memory metrics showed a clear **sawtooth** — climbing to the 512Mi limit then dying. That's a **leak**, not an under-provisioned limit. We raised the limit as a stopgap, alerted at 85%, and the dev team fixed the leak (an unbounded cache). Restarts stopped.

### Key takeaway

> “`OOMKilled` = container exceeded its **memory limit**. Confirm with `kubectl describe` + memory metrics, then distinguish **limit-too-low** (raise requests/limits) from a **memory leak** (rising sawtooth → fix the app). Add a **memory-usage alert** so it's caught early.”
