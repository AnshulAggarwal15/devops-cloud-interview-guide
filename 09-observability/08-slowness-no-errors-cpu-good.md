## Users Report Slowness, but Logs Show No Errors and CPU is Fine — How Do You Fix It?

### Question  
Users report the app is slow. Logs don't show errors, and CPU utilization is healthy. How will you fix it?

### Short explanation of the question  
This is a classic observability scenario. "No errors + healthy CPU" rules out the obvious causes, so the interviewer wants a **methodical, telemetry-driven** approach — using latency metrics and traces to find where time is spent (often a downstream dependency, DB, or lock).

### Answer  
I stop guessing and follow the telemetry: check **latency metrics (p95/p99)** to confirm and localize the slowness, then use **distributed traces** to find which service or call is slow. Slow-but-error-free usually means **waiting** — a slow DB query, a downstream API, lock contention, or resource saturation other than CPU (memory, disk I/O, connection pools).

### Detailed explanation of the answer for readers’ understanding

#### 1. 📊 Confirm and localize with latency metrics
- Look at **p95/p99 latency** per service/endpoint, not averages.
- Identify *which* endpoint/service is slow and *when* it started.

#### 2. 🔗 Follow a trace
- Open **distributed traces** (Jaeger/Tempo/X-Ray) for slow requests.
- The trace breaks latency down by **span** — you'll see the slow hop (DB call, external API, queue wait).

#### 3. 🔎 Check the "waiting", not the "working"
CPU is fine because the app is **blocked/waiting**, not computing. Look at:

| Suspect | How to check |
|---|---|
| **Slow DB query** | DB metrics, slow-query log, trace span on the query |
| **Downstream API** | Trace span latency to the dependency |
| **Lock/contention** | Thread dumps, DB locks, connection pool waits |
| **Connection pool exhaustion** | Pool metrics (waiting threads) |
| **Disk/network I/O** | Node I/O saturation, network latency |
| **GC pauses** | JVM GC metrics/logs |

#### 4. 🛠️ Fix the specific cause
- Add an index / optimize the query, add caching, increase pool size, add a timeout+retry to a slow dependency, or scale the saturated resource.

---

### 🧠 Real-World Example

> Latency p99 was high but CPU and error rate were normal. A **trace** showed 90% of the time was a single DB call. The DB's slow-query log revealed a missing index. Adding the index cut p99 dramatically — no code errors were ever logged because nothing *failed*, it just **waited**.

### Key takeaway

> “Slow with no errors and healthy CPU means the app is **waiting**, not failing. Use **latency metrics (p95/p99)** to localize and **traces** to find the slow span — usually a DB query, downstream API, lock, or pool exhaustion — then fix that specific bottleneck.”
