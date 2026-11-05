## Question  
What kind of metrics do you scrape with Prometheus in your current organization?

### 📝 Short Explanation  
An experience question. The interviewer wants to hear that you scrape a **sensible mix** across layers — infrastructure, Kubernetes, application (RED), and business metrics — not just "CPU and memory".

## ✅ Answer  
I scrape metrics across four layers: **infrastructure** (node CPU/memory/disk via node_exporter), **Kubernetes** (pod/deployment state via kube-state-metrics & cAdvisor), **application** (request rate, errors, latency — the RED method), and a few **business** metrics (orders, signups). Alerts and SLOs are built on these.

### 📘 Detailed Explanation  

#### 🧱 The layers I scrape

| Layer | Examples | Source |
|---|---|---|
| **Infrastructure** | CPU, memory, disk, network, filesystem | **node_exporter** |
| **Kubernetes** | Pod restarts, phase, deployment replicas, resource usage | **kube-state-metrics**, **cAdvisor** |
| **Application** | Request rate, error rate, latency (p50/p95/p99) | App instrumentation (client libs) |
| **Middleware/DB** | Queue depth, DB connections, cache hit rate | Exporters (postgres/redis exporter) |
| **Business** | Orders/sec, signups, checkout success | Custom counters |

#### 📏 The methods I follow
- **RED** (for services): **Rate**, **Errors**, **Duration**.
- **USE** (for resources): **Utilization**, **Saturation**, **Errors**.
- These give a consistent way to decide *what* is worth scraping and alerting on.

#### 🔔 What I actually alert on
- SLO-based: error rate and latency budget burn.
- Saturation: CPU/memory/disk approaching limits, pod `OOMKilled`/`CrashLoopBackOff`.
- Capacity: node pressure, PVC nearly full.

#### ⚠️ What I avoid
- Scraping high-cardinality labels (per-user IDs) that explode storage.

---

### 🧠 Real-world Example

> For our services we exposed **RED** metrics (request rate, 5xx rate, latency histogram), scraped node metrics via **node_exporter**, and Kubernetes state via **kube-state-metrics**. Our top alerts were SLO burn (latency/errors), pod restarts, and disk saturation — the signals that actually predicted incidents.

> Summary:  
> Scrape a **layered mix**: infra (node_exporter), Kubernetes (kube-state-metrics/cAdvisor), app **RED** metrics (rate/errors/duration), middleware exporters, and key business metrics — following **RED/USE** and avoiding high-cardinality label explosions.
