## How Do You Trace a Request Across Multiple Microservices in a Kubernetes Cluster?

### Question  
How do you trace a single request as it flows across multiple microservices in a Kubernetes cluster?

### Short explanation of the question  
This is about **distributed tracing**. In microservices, one user request touches many services, so the interviewer wants to know about **trace context propagation**, instrumentation (OpenTelemetry), and a tracing backend (Jaeger/Tempo).

### Answer  
I use **distributed tracing**: each request gets a unique **trace ID** that is **propagated** across services via headers (W3C `traceparent`). Services are instrumented with **OpenTelemetry** to create **spans**, which are exported to a backend like **Jaeger** or **Tempo**, where I can see the full request path and per-service latency.

### Detailed explanation of the answer for readers’ understanding

#### 1. 🔗 Trace context propagation
- The first service generates a **trace ID** (and span ID).
- It passes them downstream in **HTTP headers** — the **W3C Trace Context** standard (`traceparent`), or B3 headers.
- Every service continues the same trace, creating child **spans**, so all the work is stitched into one trace.

#### 2. 🛠️ Instrumentation with OpenTelemetry
- Add the **OpenTelemetry SDK** (auto-instrumentation exists for common frameworks/HTTP clients).
- It creates spans automatically for inbound/outbound calls and propagates headers for you.

```text
Span: order-svc  ─┬─> Span: payment-svc ──> Span: bank-api
                  └─> Span: inventory-svc ──> Span: db-query
```

#### 3. 📤 Export to a backend
- Spans go to the **OTel Collector**, which exports to **Jaeger** or **Grafana Tempo**.
- In Kubernetes, run the Collector as a DaemonSet/Deployment; apps send spans to it.

#### 4. 🔍 Analyze
- In Jaeger/Tempo, open a slow trace to see **which span** dominates latency and where an error originated.
- Correlate the **trace ID** with logs (log the trace ID) to jump straight to the relevant log lines.

---

### 🧠 Real-World Example

> A checkout was intermittently slow. With **OpenTelemetry** propagating `traceparent` across services and **Tempo** as the backend, one trace showed the delay was in the `inventory-svc → db` span. We also logged the **trace ID**, so we jumped from the trace directly to that service's logs and found the slow query.

### Key takeaway

> “Distributed tracing = a **trace ID propagated via headers** (W3C `traceparent`) with each service emitting **spans** through **OpenTelemetry**, exported to **Jaeger/Tempo**. It reveals the full request path and the slow/failing hop — correlate the trace ID with logs for root cause.”
