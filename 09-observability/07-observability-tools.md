## Question  
Which tools have you used to build an observability stack?

### 📝 Short Explanation  
An experience question. The interviewer wants to hear tools grouped by the **three pillars** (metrics, logs, traces) plus **visualization and alerting** — and ideally why you picked them.

## ✅ Answer  
I've built stacks with **Prometheus + Grafana** for metrics and dashboards, **Loki** or the **ELK stack** for logs, **Jaeger/Tempo** for traces, **Alertmanager** for alerting, and **OpenTelemetry** to instrument apps in a vendor-neutral way. On AWS I've also used **CloudWatch** and **X-Ray**.

### 📘 Detailed Explanation  

| Pillar / function | Tools I've used |
|---|---|
| **Metrics** | Prometheus, VictoriaMetrics, CloudWatch Metrics |
| **Dashboards** | **Grafana** |
| **Logs** | Loki, ELK (Elasticsearch/Logstash/Kibana), Fluent Bit/Fluentd (shippers), CloudWatch Logs |
| **Traces** | Jaeger, Grafana Tempo, AWS X-Ray |
| **Instrumentation** | **OpenTelemetry** (SDK + Collector), Prometheus client libs |
| **Alerting** | Alertmanager, PagerDuty, Opsgenie (routing/on-call) |

#### 🧱 A typical stack I've assembled
- **Prometheus** scrapes metrics (via Kubernetes service discovery) → **Grafana** dashboards.
- **Fluent Bit** ships container logs → **Loki** (queryable in Grafana next to metrics).
- **OpenTelemetry** traces → **Tempo/Jaeger**.
- **Alertmanager** routes SLO-based alerts to **Slack/PagerDuty**.

#### 🧭 Why these
- Prometheus + Grafana + Loki + Tempo is a cohesive, open-source stack that shares one UI (Grafana).
- **OpenTelemetry** avoids vendor lock-in — you can swap backends later.
- On AWS-only shops, **CloudWatch + X-Ray** reduce operational overhead.

---

### 🧠 Real-world Example

> We ran the **Grafana stack** — Prometheus (metrics), Loki (logs), Tempo (traces) — all viewable in Grafana, with **Alertmanager → PagerDuty**. Apps were instrumented with **OpenTelemetry**, so when we later evaluated a managed backend, we didn't have to re-instrument anything.

> Summary:  
> Group by pillar: **Prometheus + Grafana** (metrics/dashboards), **Loki/ELK** (logs), **Jaeger/Tempo/X-Ray** (traces), **Alertmanager/PagerDuty** (alerting), and **OpenTelemetry** for vendor-neutral instrumentation. On AWS, **CloudWatch + X-Ray** are the managed options.
