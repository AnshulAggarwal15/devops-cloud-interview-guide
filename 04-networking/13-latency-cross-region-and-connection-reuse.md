## Reducing Latency Across Regions — Keep-Alive, Connection Reuse & Edge Termination

### Question  
Users in India hit an API in **US-East** and see high latency. **Keep-alive** helped some users but not new ones. If new users keep arriving, how do you actually fix it?

### Short explanation of the question  
This is a **physics + architecture** question. Keep-alive only helps clients that **reuse an existing connection** — every **new** client still pays a full **TCP + TLS handshake** across a long-distance link (each round trip ≈ the India→US-East RTT, ~200+ ms). You can't change the speed of light, so the answer is two-pronged: **reduce the cost/number of new connections**, and **move the connection endpoint closer to users**.

---

### Answer  
- **Keep-alive only helps reused connections.** New clients still do TCP + TLS setup, and each handshake round trip costs the full cross-region RTT.
- **Reduce per-connection cost:** **HTTP/2 or HTTP/3** (multiplex many requests over one connection), **TLS 1.3 + session resumption / 0-RTT** (fewer handshake round trips), and **connection pooling** for clients making multiple calls.
- **Move the endpoint closer (the real fix for RTT):** a **CDN / edge** that **terminates TLS near the user** and keeps a warm, persistent connection back to US-East; or **multi-region / regional endpoints** with geo/latency routing so Indian users hit an ap-south endpoint.
- **Key point to state:** if the root cause is **physical RTT**, no application tweak removes it — you must **terminate connections closer to users or serve from a closer region**. Protocol/TLS tuning only reduces *handshake overhead*.

---

### Detailed explanation of the answer for readers’ understanding

### 🧭 Why keep-alive isn't enough
- **Keep-alive** keeps a TCP connection open so subsequent requests **skip the handshake** — great for a client that's already connected.
- But a **new** client (or a client whose connection expired) must do **TCP (1 RTT) + TLS (1–2 RTT)** before the first byte. Over India→US-East (~200+ ms one way effects), that's **hundreds of ms of pure setup** before any data.
- With continuous new users, you're constantly paying that setup cost — keep-alive can't help connections that don't exist yet.

### 🛠️ Lever 1 — cut the cost/number of new connections
| Technique | What it saves |
|---|---|
| **HTTP/2 / HTTP/3 multiplexing** | Many requests share **one** connection → far fewer handshakes |
| **TLS 1.3** | Handshake in **1-RTT** (vs 2) for new connections |
| **TLS session resumption / 0-RTT** | Returning clients **skip** most of the handshake |
| **HTTP/3 (QUIC)** | Combined transport+crypto setup, **0-RTT** resume, survives network changes |
| **Connection pooling** | Clients reuse warm connections instead of dialing per request |
These **reduce handshake overhead**, but they **don't remove the base RTT** for the data itself.

### 🌍 Lever 2 — move the endpoint closer (fixes the RTT)
This is the architectural fix when distance is the root cause:
- **CDN / edge termination (CloudFront, etc.):** the user connects to a **nearby edge PoP** and does the TCP/TLS handshake **there** (short RTT). The edge holds a **warm, reused** connection back to US-East, so the expensive long-haul handshake happens **once**, not per user. Great for cacheable content *and* as a TLS-terminating front door for APIs.
- **Multi-region / regional endpoints:** deploy the API in **ap-south-1** too and use **Route 53 latency/geo routing** so Indian users hit the closer region. Removes the long-haul RTT entirely (at the cost of running/consistency across regions — data replication becomes the hard part).
- **Global accelerator-style anycast:** get users onto AWS's backbone at a nearby edge, then traverse AWS's network (more stable/faster than the public internet) to US-East.

### 🧪 The diagnosis-first framing (say this)
> "First I'd confirm **what's actually causing the latency** — is it handshake/connection setup, server processing, or pure network RTT? Measure it (client timings, TTFB, `curl -w`, tracing). If it's **connection setup**, HTTP/2-3 + TLS 1.3 + pooling help. If it's **physical distance**, protocol tuning won't fix it — I terminate at the **edge** or serve from a **closer region**."

### ⚠️ Things they probe
- **Keep-alive helps only reused connections; new clients still handshake.**
- Protocol/TLS tuning reduces **handshake overhead**, not the **base RTT** — name that distinction.
- The real RTT fix = **edge TLS termination (CDN) or multi-region + latency routing**.
- Diagnose before prescribing: **setup cost vs processing vs network distance**.

---

### 🧪 Real-World Example

> "Indian users had ~400 ms API latency to US-East, worst for first-time callers. Keep-alive only helped repeat callers. We put **CloudFront** in front: users did TLS **at a Mumbai edge** (short RTT) while the edge reused a warm connection to US-East, and enabled **HTTP/2 + TLS 1.3**. First-byte latency dropped sharply. Where the workload justified it, we later added an **ap-south-1 regional endpoint** with Route 53 latency routing to remove the long-haul entirely."

---

### Key takeaway

> "**Keep-alive only helps reused connections** — new clients still pay TCP+TLS setup at full cross-region RTT. Reduce that with **HTTP/2-3 multiplexing, TLS 1.3 + session resumption, and connection pooling**; but if the root cause is **physical distance**, the only real fix is to **terminate connections at the edge (CDN) or serve from a closer region** with latency-based routing. Diagnose setup-cost vs distance first."

### Related
- [HTTP/1.1 vs 2 vs 3 & multiplexing](./11-http-1-2-3-and-multiplexing.md) · [TLS handshake & versions](./12-tls-handshake-and-versions.md)
- [Slowness in an app](./04-slowness-in-app.md) · [Route 53 routing (latency/geo)](../10-aws/112-route53-dns-routing.md) · [CloudFront/CDN in cloud security scenarios](../10-aws/28-cloud-security-scenarios.md)
