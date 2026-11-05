## HTTP/1.1 vs HTTP/2 vs HTTP/3 (and Multiplexing)

### Question  
What's the difference between **HTTP/1.1, HTTP/2, and HTTP/3**? And what exactly is **multiplexing**?

### Short explanation of the question  
Compare them on three axes: **transport**, **multiplexing**, and **head-of-line (HOL) blocking**. HTTP/1.1 = one request at a time per connection (TCP). HTTP/2 = many requests concurrently over one TCP connection (multiplexing). HTTP/3 = same multiplexing but over **QUIC/UDP**, removing TCP-level HOL blocking. Multiplexing = **many requests/responses sharing one connection at the same time**.

---

### Answer  
- **HTTP/1.1** — text protocol over **TCP**; one request/response per connection at a time (keep-alive reuses the connection, but concurrency needs **multiple** connections). Prone to **HOL blocking** at the request level.
- **HTTP/2** — still **TCP**, but adds **multiplexing** (many concurrent streams over one connection) + **HPACK header compression** + server push. Removes app-level HOL blocking, but **TCP-level** HOL blocking remains (one lost packet stalls all streams).
- **HTTP/3** — moves transport to **QUIC over UDP**, keeping multiplexing but making streams **independent** (a lost packet only stalls its own stream), with **TLS 1.3 built in** and faster connection setup + connection migration.
- **Multiplexing** = carrying **multiple independent requests/responses concurrently over a single connection** (via streams/frames), so you don't pay a new handshake per request.

---

### Detailed explanation of the answer for readers’ understanding

### 🔀 Side by side
| | HTTP/1.1 | HTTP/2 | HTTP/3 |
|---|---|---|---|
| Transport | TCP | TCP | **QUIC over UDP** |
| Multiplexing | ❌ (1 req/conn at a time) | ✅ streams | ✅ streams |
| Header compression | none/basic | **HPACK** | **QPACK** |
| HOL blocking | at request level | app-level solved; **TCP-level remains** | **avoids transport HOL blocking** |
| TLS | usually on top | usually on top | **TLS 1.3 integrated** |
| Setup cost | TCP (+TLS) per new conn | TCP+TLS once, reused | **faster** (QUIC combines transport+crypto), 0-RTT resume |

### 🧵 What multiplexing actually means
Without it (HTTP/1.1), concurrency means opening **several TCP connections** (browsers cap ~6 per host), each paying its own handshake.
With HTTP/2, one connection carries many **streams**; requests are split into **frames** that interleave:
```text
             ONE TCP+TLS connection
Client  ├── Stream 1  → Request A ─┐
        ├── Stream 3  → Request B ─┼─ frames interleaved, concurrent
        └── Stream 5  → Request C ─┘
```
So five API calls don't need five handshakes — they share one connection concurrently. That's the win: **less connection/handshake overhead, especially for many small requests**.

### 🚧 Head-of-line (HOL) blocking — the nuance
- **HTTP/1.1:** a slow response blocks the ones behind it on that connection.
- **HTTP/2:** solves it at the HTTP layer (streams are independent *logically*)… but they all ride **one TCP stream**, so if a **TCP packet is lost**, TCP must wait for retransmission and **every** stream stalls — **transport-level HOL blocking**.
- **HTTP/3 / QUIC:** streams are **independent at the transport level**, so a lost packet only affects **its** stream; the rest keep flowing. This is the core reason HTTP/3 exists.

### ⚡ Why HTTP/3 / QUIC is faster to connect
- QUIC merges the **transport + TLS 1.3** handshake, so a new connection needs **fewer round trips** (and **0-RTT** for resumed connections).
- **Connection migration**: a QUIC connection has a connection ID, so it survives a network change (Wi-Fi → cellular, IP change) without a full re-handshake.

### ⚠️ Things they probe
- **Multiplexing = many requests over one connection concurrently** (streams/frames) — the headline HTTP/2 feature.
- HTTP/2 still suffers **TCP-level HOL blocking**; **HTTP/3 fixes it via QUIC/UDP**.
- HTTP/3 = **QUIC over UDP + TLS 1.3 integrated**, faster setup, connection migration.
- Header compression: **HPACK (h2) / QPACK (h3)**.

---

### 🧪 Real-World Example

> "A client made lots of small API calls. On HTTP/1.1 the browser opened ~6 connections, each with its own TCP+TLS handshake. Enabling **HTTP/2** let all calls share **one** multiplexed connection — fewer handshakes, lower latency. For lossy mobile networks we looked at **HTTP/3**, since QUIC keeps one stream's packet loss from stalling the others."

---

### Key takeaway

> "**HTTP/1.1 = TCP, effectively one request per connection at a time. HTTP/2 = TCP + multiplexing (many concurrent streams over one connection) + HPACK, but still TCP-level HOL blocking. HTTP/3 = QUIC over UDP with TLS 1.3 built in, keeping multiplexing while eliminating transport HOL blocking and speeding up connection setup.** Multiplexing = multiple requests/responses sharing one connection concurrently."

### Related
- [TLS handshake & versions](./12-tls-handshake-and-versions.md) · [Cross-region latency & connection reuse](./13-latency-cross-region-and-connection-reuse.md)
- [DNS](./01-explain-dns.md) · [OSI model](./02-explain-osi-model-with-example.md)
