## Question  
What is the difference between logs, metrics, and traces?

### 📝 Short Explanation  
These are the **three pillars of observability**. Each answers a different question, and interviewers want to know you understand when to use which — and that they complement each other.

## ✅ Answer  
- **Logs** = discrete, timestamped **event records** ("what happened").
- **Metrics** = numeric **measurements over time**, aggregated ("how much / how many").
- **Traces** = the **end-to-end path of a single request** across services ("where did the time go").

### 📘 Detailed Explanation  

| | Logs | Metrics | Traces |
|---|---|---|---|
| What | Event records | Numeric time-series | Request journey across services |
| Answers | *What happened?* | *How much / trend?* | *Where is the bottleneck?* |
| Cardinality | High detail | Low, aggregated | Per-request spans |
| Cost | Can be large | Cheap to store | Moderate; often sampled |
| Tools | ELK, Loki, CloudWatch Logs | Prometheus, CloudWatch | Jaeger, Tempo, X-Ray |

#### 📜 Logs
Detailed records of individual events — errors, requests, state changes.
```
2026-08-01T10:02:11Z ERROR order-svc payment failed order_id=123 provider=stripe
```
Great for **root-cause detail**, but expensive to store and hard to aggregate alone.

#### 📊 Metrics
Aggregated numbers over time — request rate, error rate, latency, CPU.
- Cheap, ideal for **dashboards and alerts** (SLOs, RED/USE method).
- Low cardinality; you can't store a label per user without blowing up storage.

#### 🔗 Traces
Follow **one request** as it flows through multiple services, breaking down time per **span**.
- Perfect for **microservices** — shows which hop is slow.
- Uses context propagation (trace ID) across service boundaries.

#### 🧩 They work together
Metric alert fires → traces show which service is slow → logs of that service explain why.

---

### 🧠 Real-world Example

> A **metric** alert flagged rising p99 latency. A **trace** showed the delay was in the inventory service's DB call. The inventory service's **logs** revealed slow queries from a missing index. All three pillars, each doing its job.

> Summary:  
> **Logs = detailed events (what)**, **metrics = aggregated numbers/trends (how much)**, **traces = per-request path across services (where)**. Use metrics to alert, traces to localize, and logs to find the root cause.
