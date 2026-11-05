## Question  
What is the difference between monitoring and observability?

### 📝 Short Explanation  
These terms are used interchangeably but mean different things. **Monitoring** tells you *whether* something is wrong (known problems); **observability** helps you understand *why* it's wrong (including unknown problems) by exploring your system's outputs.

## ✅ Answer  
- **Monitoring** is watching **predefined metrics and alerts** for **known failure modes** — "is CPU high?", "is the service up?".
- **Observability** is the ability to **ask new questions** about your system from its **logs, metrics, and traces** — so you can debug problems you didn't anticipate.

Monitoring is a **subset** of observability.

### 📘 Detailed Explanation  

| | Monitoring | Observability |
|---|---|---|
| Answers | *Is it broken?* (known-unknowns) | *Why is it broken?* (unknown-unknowns) |
| Approach | Predefined dashboards & alerts | Explore/ask ad-hoc questions |
| Built on | Metrics, alerts | **Logs + metrics + traces** |
| Example | "Alert if 5xx > 1%" | "Which service, which endpoint, which users, and why?" |

#### 🧠 The key idea
- **Monitoring** is great for **things you already know can fail** — you set thresholds and get paged.
- **Observability** is about **understanding the system deeply enough** to investigate **novel** failures without shipping new code first. You get there by emitting rich telemetry (the three pillars: logs, metrics, traces).

#### 🔧 In practice
- You still need monitoring (alerts on SLOs, error rates, saturation).
- You add observability by instrumenting apps so you can slice by dimensions (endpoint, region, version) and follow a request across services.

---

### 🧠 Real-world Example

> Monitoring told us "checkout latency is high" (a known alert). But the dashboard didn't say *why*. Because we had **observability** — traces and high-cardinality metrics — we drilled in and found only requests hitting one payment provider in one region were slow. Monitoring raised the flag; observability found the cause.

> Summary:  
> **Monitoring = detecting known problems** via predefined metrics/alerts. **Observability = understanding the system** (via logs, metrics, traces) well enough to debug **unknown** problems. Monitoring is a subset of observability.
