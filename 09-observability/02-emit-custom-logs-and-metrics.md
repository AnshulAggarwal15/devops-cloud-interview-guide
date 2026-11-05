## Question  
How do you emit custom logs and metrics in your application?

### 📝 Short Explanation  
This checks whether you know how applications are **instrumented** — using logging libraries for structured logs and client libraries (or OpenTelemetry) to expose custom metrics that a system like Prometheus can collect.

## ✅ Answer  
For **logs**, I use a structured logging library (JSON logs) writing to stdout so the platform ships them to a log backend. For **metrics**, I instrument the code with a **Prometheus client library** (or **OpenTelemetry**) to expose counters, gauges, and histograms on a `/metrics` endpoint that Prometheus scrapes.

### 📘 Detailed Explanation  

#### 📜 Custom logs
- Use a logging library (e.g. `logging` in Python, `logback`/`slf4j` in Java, `winston` in Node).
- Emit **structured (JSON) logs** with context fields (request_id, user, service, level).
- Write to **stdout/stderr** — in containers/Kubernetes the platform (Fluent Bit/Fluentd) collects and ships them to Loki/ELK/CloudWatch.

```python
import logging, json_log_formatter
logger = logging.getLogger("orders")
logger.info("order_created", extra={"order_id": 123, "amount": 40.0})
```

#### 📊 Custom metrics (Prometheus client)
Instrument the app to expose metrics on `/metrics`:
```python
from prometheus_client import Counter, Histogram, start_http_server

orders_total = Counter("orders_total", "Total orders", ["status"])
latency = Histogram("order_latency_seconds", "Order processing latency")

@latency.time()
def process_order(...):
    orders_total.labels(status="success").inc()

start_http_server(8000)   # exposes /metrics
```

#### 🔧 Metric types
| Type | Use |
|---|---|
| **Counter** | Monotonic count (requests, errors) |
| **Gauge** | Value up/down (queue depth, temperature) |
| **Histogram** | Distributions (latency buckets → p95/p99) |
| **Summary** | Client-side quantiles |

#### 🌐 OpenTelemetry (vendor-neutral)
For a standard approach, use the **OpenTelemetry SDK** to emit logs, metrics, and traces, and export via the **OTel Collector** to any backend.

---

### 🧠 Real-world Example

> We added a `payment_failures_total` **counter** labeled by provider and a **histogram** for checkout latency. Prometheus scraped `/metrics`, and we alerted on the failure rate and graphed p99. Structured JSON logs to stdout were shipped by Fluent Bit to Loki for detail.

> Summary:  
> **Logs:** structured (JSON) via a logging library to stdout, shipped by the platform. **Metrics:** instrument with a **Prometheus client** (Counter/Gauge/Histogram) exposed on `/metrics`, or use **OpenTelemetry** for a vendor-neutral logs+metrics+traces pipeline.
