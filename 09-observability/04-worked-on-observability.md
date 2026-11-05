## Question  
Have you worked on observability? If yes, explain what you did.

### 📝 Short Explanation  
An experience question. The interviewer wants a concrete story: what stack you built, what you instrumented, what alerts/dashboards you created, and the **outcome** (faster incident resolution, fewer false alarms).

## ✅ Answer  
Yes. I built an observability stack around **Prometheus + Grafana** for metrics, **Loki/ELK** for logs, and **Jaeger/Tempo** for traces. I instrumented services with **RED metrics**, created SLO dashboards and alerts in **Alertmanager**, and added distributed tracing so we could pinpoint slow services quickly.

### 📘 Detailed Explanation  

#### 🛠️ What I set up
- **Metrics:** Prometheus scraping node_exporter, kube-state-metrics, and app `/metrics`; visualized in **Grafana**.
- **Logs:** structured JSON logs shipped by **Fluent Bit** to **Loki** (or ELK), queryable alongside metrics.
- **Traces:** instrumented services with **OpenTelemetry**, exported to **Jaeger/Tempo** for end-to-end request views.
- **Alerting:** **Alertmanager** with routing to Slack/PagerDuty, based on **SLOs** rather than raw thresholds.

#### 📊 What I instrumented
- **RED** for each service (rate, errors, duration).
- Saturation metrics (CPU/mem/disk), pod health (restarts, OOMKilled).
- A few business KPIs (checkout success rate).

#### 🎯 The outcome
- **Faster MTTR:** metric alert → trace to the slow service → logs for root cause.
- **Fewer false alarms:** moved from noisy static thresholds to SLO/burn-rate alerts.
- Shared **Grafana dashboards** gave teams self-service visibility.

#### 🧭 Framework I followed
- **RED/USE** methods for what to measure.
- **SLIs/SLOs** to decide what's worth paging on.

---

### 🧠 Real-world Example

> On my last team, incidents took too long to diagnose. I stood up **Prometheus + Grafana + Loki**, instrumented services with **RED** metrics and **OpenTelemetry tracing**, and rebuilt alerts around **SLO burn rate**. Result: on-call could go metric → trace → log in minutes instead of guessing, and pages dropped because we alerted on symptoms that mattered.

> Summary:  
> Tell a concrete story: built **Prometheus/Grafana + Loki/ELK + Jaeger/Tempo**, instrumented **RED metrics** and **tracing**, alerted on **SLOs** via Alertmanager. Emphasize the **outcome** — faster MTTR and fewer false alarms.
