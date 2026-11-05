## Central Observability for 20–30 Kubernetes Clusters

### Question  
You run 20–30 Kubernetes clusters and don't want to install Grafana on each one. Lay out a simple **central observability** architecture: one Grafana that can see pods/nodes/CoreDNS across every cluster. Where does Prometheus live? What's Grafana's data source? And how do you tell **which metric came from which cluster**?

### Short explanation of the question  
This is a **fan-in** architecture question. The interviewer wants: **collection per cluster**, **one central store that consolidates**, **a single Grafana** pointed at the *consolidated* store (not at 20 Prometheis), and — the detail that trips people up — **how metrics are labeled per cluster** (`external_labels`).

---

### Answer  
- **Per cluster**: a lightweight **Prometheus (agent mode)** scrapes local metrics and **`remote_write`s** them out. Grafana is **not** installed per cluster.
- **Central cluster**: a **consolidation layer** — **Thanos / Grafana Mimir / Cortex** (or a Prometheus configured as a remote-write receiver) backed by **object storage (S3)** — that ingests from all clusters.
- **Grafana**: one instance in the central cluster, with a **single data source = the central query layer** (Thanos Query / Mimir), **not** each cluster's Prometheus.
- **Which metric is which cluster**: every agent stamps a unique **`external_labels: { cluster: "a", region: "..." }`**, so all metrics carry a `cluster` label you filter/group by in Grafana.

---

### Detailed explanation of the answer for readers’ understanding

### 🏗️ The architecture (fan-in)
```text
Cluster A ─ Prometheus (agent) ─┐
Cluster B ─ Prometheus (agent) ─┤ remote_write
Cluster C ─ Prometheus (agent) ─┼──────────────►  Central cluster
   ...            (adds          │                 ┌───────────────────────────┐
Cluster N ─ Prometheus (agent) ─┘  external        │  Mimir / Thanos-Receive   │
                                    labels)         │  + Object storage (S3)    │
                                                    │  Thanos Query / Mimir     │
                                                    └────────────┬──────────────┘
                                                                 │ single data source
                                                                 ▼
                                                            Grafana (one)
```

### 🔻 Per-cluster collector
- Run **Prometheus in agent mode** (or a full Prometheus with short local retention). It scrapes node-exporter, kube-state-metrics, cAdvisor/kubelet, CoreDNS, kube-proxy, and app metrics, then **`remote_write`s** to the central endpoint.
- Agent mode is cheap: no local querying/long retention — it just forwards. This is why you don't put Grafana here.

### 🎯 Central consolidation (the key idea)
- The central cluster runs a **remote-write receiver + long-term store**: **Grafana Mimir**, **Cortex**, or **Thanos** (Receive or per-cluster Sidecars → Store), all backed by **S3/GCS** for cheap, scalable retention.
- Grafana talks to the **query front-end of that layer** (Thanos **Querier** / Mimir query frontend). That single endpoint **consolidates all clusters**, so Grafana has **one data source** — not 20.
- Why not point Grafana at each cluster's Prometheus directly? It technically works (mixed data sources), but it doesn't scale, breaks cross-cluster queries, and every cluster must be network-reachable and highly available. **Central consolidation is the right design.**

### 🏷️ Telling clusters apart — `external_labels`
Each agent adds identifying labels at the source:
```yaml
# prometheus.yml on cluster A
global:
  external_labels:
    cluster: cluster-a
    region: ap-south-1
    env: prod
```
- Now every series from cluster A carries `cluster="cluster-a"`. Node, pod, CoreDNS, kube-proxy metrics — all tagged.
- In Grafana you filter/group by `cluster` (a **template variable**), so one dashboard shows any cluster or all of them.
- **Careful:** the label set (`cluster`, maybe `region`) must be **unique per cluster**, or series from different clusters collide/overwrite. Thanos/Mimir also use these labels for dedup and for routing queries.

### 🔎 Logs too (ELK / Loki)
The same pattern applies to logs: ship per cluster (Fluent Bit) → central **Loki** or **ELK**, and stamp a `cluster` field. On retention questions (e.g. "ELK keeps 30 days"), know the levers: **retention window, index/shard sizing, hot-warm-cold tiers**, and that **disk = ingest rate × retention × replication** (see below).

### ⚠️ Things they probe
- "Grafana talks to 20 Prometheis or to one consolidator?" → **one consolidator** (Thanos Query / Mimir). That's the whole point of central observability.
- "How do you distinguish clusters?" → **`external_labels` (cluster/region)**; use them as Grafana template variables and for Thanos/Mimir dedup.
- Name **Thanos vs Mimir vs Cortex** as the consolidation options; **remote_write** as the transport.

---

### 🧪 Real-World Example

> “We had ~25 clusters. Each ran **Prometheus in agent mode** with `external_labels: {cluster, region}` and `remote_write` to a central **Mimir** cluster backed by **S3**. One **Grafana** used Mimir as its **single data source**; a `cluster` template variable let us drill into any cluster or view the whole fleet. Adding a new cluster was just deploying the agent with a new `cluster` label.”

---

### Key takeaway

> “**Collect per cluster (Prometheus agent + `remote_write`), consolidate centrally (Thanos/Mimir/Cortex on S3), visualize once (single Grafana pointed at the consolidator).** Stamp every cluster with unique **`external_labels` (cluster/region)** so you always know which metric came from which cluster.”

### Related
- [Observability tools](./07-observability-tools.md)
- [Metrics to scrape with Prometheus](./03-metrics-to-scrape-with-prometheus.md)
- [Push vs pull monitoring](./06-push-vs-pull-monitoring.md)
