## The TLS Handshake, and TLS 1.2 vs 1.3

### Question  
What happens during a **TLS handshake**? What's the difference between **TLS 1.2 and 1.3**, and why is 1.3 better?

### Short explanation of the question  
The handshake is: **negotiate → authenticate → key exchange → derive session keys → encrypted data**. The single most important fact to state: **the server's private key is never sent over the network** — the client gets the **certificate (public key)**, verifies it, and both sides *derive* the same symmetric key. TLS 1.3 does this in **fewer round trips** and drops legacy crypto.

---

### Answer  
- A TLS handshake **negotiates** the version/cipher, **authenticates** the server via its **certificate**, performs a **key exchange** to agree on shared secret material, both sides **derive the same symmetric session keys**, and then **encrypted application data** flows.
- **Private keys never travel** — the client receives the **public certificate**, validates the chain/hostname, and uses the key-exchange (e.g. ECDHE) to establish the shared secret.
- **TLS 1.3 vs 1.2:** 1.3 needs **fewer round trips** (1-RTT, or **0-RTT** on resumption), **removes obsolete/weak crypto**, mandates **forward secrecy**, and simplifies cipher negotiation. Keep **1.2 + 1.3** enabled for compatibility; disable 1.0/1.1.

---

### Detailed explanation of the answer for readers’ understanding

### 🤝 The TLS 1.2 handshake (simplified)
```text
Client                                   Server
| ---- ClientHello --------------------> |   TLS versions, cipher suites, client random
| <--- ServerHello --------------------- |   chosen version/cipher, server random
| <--- Certificate --------------------- |   server's PUBLIC cert (proves identity)
| <--- ServerKeyExchange* -------------- |   key-exchange params (e.g. ECDHE)
| <--- ServerHelloDone ----------------- |
| ---- ClientKeyExchange --------------> |   client's key-exchange contribution
| ---- ChangeCipherSpec / Finished ----> |
| <--- ChangeCipherSpec / Finished ----- |
| ============ Encrypted data =========> |
```
The five things happening:
1. **Negotiation** — agree on TLS version + cipher suite.
2. **Authentication** — server presents its **certificate**; client validates the **chain + hostname** (and optionally OCSP/expiry).
3. **Key exchange** — both contribute material (ECDHE) to compute a **shared secret** — *without sending the secret or the private key*.
4. **Derive session keys** — both independently derive the **same symmetric keys** from that secret.
5. **Encrypted communication** — fast symmetric encryption (AES/ChaCha) for the actual data.

> **The point interviewers push on:** the **private key stays on the server**. It's used to prove identity / sign, never transmitted. The client only ever sees the **public** certificate. Asymmetric crypto is used *just* to bootstrap a **symmetric** session key.

### ⚡ TLS 1.3 — what changed
```text
Client                                   Server
| ---- ClientHello (+ key share) ------> |
| <--- ServerHello + Certificate + ----- |
|      Finished (keys already agreed)     |
| ---- Finished ----------------------> |
| ============ Encrypted data =========> |   ← 1 round trip
```
- **Fewer round trips:** ~**1-RTT** to first byte (vs 2-RTT in 1.2), and **0-RTT** for resumed sessions — big latency win.
- **Removed legacy crypto:** no RSA key exchange, no static DH, no RC4/CBC/SHA-1 — only **AEAD** ciphers and **(EC)DHE**, so **forward secrecy is mandatory**.
- **Simpler negotiation:** far fewer cipher combinations → smaller attack surface (this is why "more secure" is really "*less* bad crypto," not just "stronger ciphers").
- **Encrypted handshake:** more of the handshake (incl. certificate) is encrypted.

### 🔁 Session resumption (why it matters for latency)
- **Resumption** (session tickets / PSK) lets a returning client skip the full handshake — TLS 1.3 supports **0-RTT** resumption, sending data on the first flight. Great for clients reconnecting frequently. (0-RTT has a replay caveat — use only for idempotent/safe requests.)

### 🧱 TLS 1.2 vs 1.3, and versions to enable
| | TLS 1.1 | TLS 1.2 | TLS 1.3 |
|---|---|---|---|
| Status | **Deprecated** | Widely used | **Preferred** |
| Handshake RTT | 2 | 2 | **1 (0 on resume)** |
| Crypto | legacy | strong suites | **AEAD-only, forward secrecy mandatory** |
| Enable in prod? | ❌ | ✅ | ✅ |
- **Nginx:** `ssl_protocols TLSv1.2 TLSv1.3;` — keep both for compatibility, drop 1.0/1.1.
- **1.3's main limitation** is **compatibility** with very old clients/libraries/middleboxes — hence dual-enabling 1.2 + 1.3.

### ⚠️ Things they probe
- **Private key is never sent** — client verifies the **public cert**, and both **derive** the symmetric key.
- Flow: **negotiate → authenticate → key exchange → derive session keys → encrypted data.**
- TLS 1.3 = **fewer round trips + removed legacy crypto + mandatory forward secrecy**; keep **1.2 for compatibility**.
- Asymmetric (handshake) just bootstraps **symmetric** (bulk data) encryption.

---

### 🧪 Real-World Example

> "For a latency-sensitive API we enabled **TLS 1.3** so new connections needed **one** round trip instead of two, and turned on **session resumption** so returning clients could reconnect with near-zero handshake cost. We kept **TLS 1.2** enabled for older clients and disabled 1.0/1.1 in Nginx."

---

### Key takeaway

> "The TLS handshake **negotiates → authenticates the server via its certificate → does a key exchange → derives symmetric session keys → then encrypts data** — and the **private key never leaves the server**. **TLS 1.3** cuts round trips (1-RTT, 0-RTT on resume), removes legacy crypto, and mandates forward secrecy; keep **TLS 1.2 + 1.3** enabled for compatibility."

### Related
- [HTTP/1.1 vs 2 vs 3 & multiplexing](./11-http-1-2-3-and-multiplexing.md) · [Cross-region latency & connection reuse](./13-latency-cross-region-and-connection-reuse.md)
- [DNS](./01-explain-dns.md) · [Authentication vs Authorization framework](../10-aws/29-authentication-vs-authorization.md)
