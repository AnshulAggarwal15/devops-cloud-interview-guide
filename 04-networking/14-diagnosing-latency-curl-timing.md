## Diagnosing Latency — DNS / TCP / TLS / TTFB with `curl` Timing

### Question  
A client in Bangalore hits an API/NLB in **US-East** and it's slow. How do you find out **which phase** is slow — DNS, TCP handshake, TLS handshake, or **TTFB** (server/backend)?

### Short explanation of the question  
The skill is **decomposing one request into measurable phases** so you can say *where* the time goes instead of guessing. **`curl -w`** gives you DNS, connect (TCP), appconnect (TLS), and starttransfer (TTFB) timings. From those you isolate **connection-establishment latency** (DNS/TCP/TLS) vs **server/backend latency** (TTFB) — and only then dig into the right layer.

---

### Answer  
Break the request into phases and measure each with **`curl -w`**:
```bash
curl -w 'DNS: %{time_namelookup}\nTCP: %{time_connect}\nTLS: %{time_appconnect}\nTTFB: %{time_starttransfer}\nTotal: %{time_total}\n' -o /dev/null -s https://api.example.com
```
- **TCP time** = `time_connect − time_namelookup`
- **TLS time** = `time_appconnect − time_connect`
- **TTFB (post-handshake)** = `time_starttransfer − time_appconnect`
- If **DNS/TCP/TLS** dominate → it's **connection setup / network distance** (fix with HTTP/2-3, TLS 1.3, reuse, edge/closer region).
- If those are normal but **TTFB** is high → the request is **into the backend path**; investigate **LB→target→app→DB/dependencies** (TTFB is *not* purely app time).

---

### Detailed explanation of the answer for readers’ understanding

### 🧩 The phases (and what each cumulative timer means)
`curl` timers are **cumulative from the start of the request**:
| curl variable | Cumulative point | Phase = difference |
|---|---|---|
| `time_namelookup` | DNS resolved | **DNS** |
| `time_connect` | TCP connected | **TCP** = `connect − namelookup` |
| `time_appconnect` | TLS established | **TLS** = `appconnect − connect` |
| `time_starttransfer` | first response byte | **TTFB** = `starttransfer − appconnect` |
| `time_total` | response complete | download = `total − starttransfer` |

```text
|--DNS--|--TCP--|--TLS--|------ wait for 1st byte (TTFB) ------|--download--|
0    namelookup connect appconnect                       starttransfer   total
```

### 🔎 How to read the result (worked example)
```text
DNS   20 ms
TCP   80 ms
TLS  100 ms      → TLS phase = 100 − 80 = 20 ms
TTFB 3.5 s       → post-handshake wait = 3.5 s − 100 ms ≈ 3.4 s
Total 3.7 s
```
- TCP+TLS are small (~100 ms); the request spends **~3.4 s waiting for the first byte** → the problem is **downstream of the handshake** (LB→target→app→DB), **not** connection setup.
- Flip it: if `TCP`/`TLS` were the big numbers and TTFB small, it's **network distance / handshake cost** → HTTP/2-3, TLS 1.3, connection reuse, or move closer (edge/region).

### 🧭 Isolate connection setup vs backend (the mental model)
```text
Client ──DNS──►──TCP──►──TLS──►  [ first byte? ]  ──► download
        └────── setup ───────┘   └──── TTFB: LB + target + app + deps ────┘
```
- **High DNS** → DNS resolver/records, not TLS/app.
- **High TCP** → network path/RTT, packet loss, congestion, no connection reuse.
- **High TLS** → handshake overhead, TLS version/cipher, no session resumption, distance (each RT costs the RTT).
- **High TTFB, setup normal** → into the backend.

### 🎯 Splitting TTFB further (NLB vs target vs app)
TTFB isn't "app time" — it includes **network to the target + backend processing**. Separate them:
- **NLB/ALB CloudWatch**: target response/processing latency, active/new connections, resets → is the LB→target hop slow, or the target itself?
- **App metrics / access logs**: request start→end timestamps, handler latency.
- **Downstream**: DB query time, external API calls.
- Conclusion pattern: *"Client→NLB 100 ms, NLB→pod 10 ms, app processing 2.8 s (DB 2.5 s) → the database is the source."*

### ⚠️ Things they probe
- `curl` timers are **cumulative** — you **subtract** to get each phase.
- **"High TTFB ≠ app is slow"** — it includes LB→target network + backend; measure target latency separately (CloudWatch) before blaming the app.
- Setup-heavy (DNS/TCP/TLS) vs backend-heavy (TTFB) points to **completely different fixes**.
- Diagnose **before** prescribing — this is the platform-engineer instinct the interviewer wanted.

---

### 🧪 Real-World Example

> "For the Bangalore→US-East slowness I ran `curl -w` from a client near the users. DNS/TCP were fine, TLS added ~20 ms, but **TTFB was ~3 s**. So it wasn't handshake cost — I moved downstream: **NLB CloudWatch** showed low target latency, but **app logs** showed a 2.5 s DB query. The fix was a DB index, not anything TLS/network. Had TCP/TLS been the big numbers, I'd have gone the other way — HTTP/2 + TLS 1.3 + edge termination."

---

### Key takeaway

> "Decompose the request with **`curl -w`** into **DNS / TCP (`connect−namelookup`) / TLS (`appconnect−connect`) / TTFB (`starttransfer−appconnect`)**. If **setup** phases dominate → network distance/handshake (HTTP/2-3, TLS 1.3, reuse, move closer). If **TTFB** dominates → the backend path — and remember **TTFB isn't pure app time**, so split **NLB target latency** (CloudWatch) from **app/DB** time before blaming the app."

### Related
- [Cross-region latency & connection reuse](./13-latency-cross-region-and-connection-reuse.md) · [TLS handshake & versions](./12-tls-handshake-and-versions.md) · [HTTP/1.1 vs 2 vs 3](./11-http-1-2-3-and-multiplexing.md)
- [Slowness in an app](./04-slowness-in-app.md) · [Slowness, no errors, CPU fine](../09-observability/08-slowness-no-errors-cpu-good.md) · [Load balancers & 502s](../10-aws/106-load-balancers-and-502.md)
