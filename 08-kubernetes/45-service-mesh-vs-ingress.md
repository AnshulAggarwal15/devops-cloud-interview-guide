## Service Mesh (Istio/Linkerd) vs Ingress Controller

### Question  
What's the difference between a **service mesh** (Istio, Linkerd) and an **ingress controller**?

### Short explanation of the question  
People conflate them because both deal with traffic. The clean split: an **ingress controller handles north-south traffic** (outside → into the cluster), while a **service mesh handles east-west traffic** (service ↔ service *inside* the cluster) and adds **mTLS, retries, and per-call observability** via **sidecar proxies**. Different problems, and you often run **both**.

---

### Answer  
- **Ingress controller** = the **front door**: routes **external** HTTP(S) into the cluster by host/path, terminates TLS. **North-south**, at the edge (NGINX, AWS LB Controller).
- **Service mesh** = the **internal traffic layer**: manages **service-to-service** calls with **sidecar proxies** (Envoy), adding **mTLS, retries/timeouts, traffic splitting, circuit breaking, and rich telemetry** — without changing app code. **East-west**, everywhere.
- They're **complementary**: ingress gets traffic *in*; the mesh governs how services talk *once inside*.

---

### Detailed explanation of the answer for readers’ understanding

### 🧭 North-south vs east-west
```text
        Users (external)
            │  north-south
            ▼
   [ Ingress Controller ]   ← edge: routing + TLS termination
            │
   ┌────────┼─────────┐  east-west (service ↔ service)
   ▼        ▼         ▼
 svc A ⇄ svc B ⇄ svc C     ← service mesh governs THESE calls
 (each pod has a sidecar proxy)
```

### 🔀 Side by side
| | Ingress Controller | Service Mesh |
|---|---|---|
| Traffic | **North-south** (in/out of cluster) | **East-west** (service↔service) |
| Scope | Cluster **edge** | **Every** pod (sidecar) or node |
| Core job | L7 routing, TLS termination | mTLS, retries, timeouts, traffic splitting, circuit breaking |
| Security | TLS at edge | **Automatic mTLS** between services, authZ policies |
| Observability | Edge metrics | **Per-call** metrics/traces for all internal traffic |
| Examples | NGINX, AWS LB Controller, Traefik | **Istio, Linkerd**, Consul |
| Cost | Low | **Operational + resource overhead** (sidecars) |

### 🕸️ How a mesh works
- A **sidecar proxy** (Envoy for Istio; a lightweight Rust proxy for Linkerd) is injected next to each app container. **All** service traffic flows through it.
- A **control plane** (istiod / Linkerd control plane) configures the proxies.
- Because the proxy is in the data path, the mesh gives you — **without touching app code**:
  - **mTLS** (encrypted + authenticated service identity) everywhere,
  - **traffic management**: canary/weighted splits, retries, timeouts, circuit breaking, fault injection,
  - **golden-signal telemetry** (latency/error/throughput) for every hop, plus distributed tracing.

### 🧩 Do you need a mesh?
- **No** for most small/medium setups — an ingress controller + NetworkPolicies + app-level retries is enough, and a mesh adds real **complexity and overhead** (a sidecar per pod).
- **Yes** when you have **many microservices** and need **uniform mTLS, fine-grained traffic control (canaries/circuit breaking), and per-call observability** you don't want to hand-code in every service. Istio = powerful/complex; **Linkerd** = lighter/simpler.
- **Gateway API** and mesh "ambient/sidecar-less" modes are blurring some lines, but the north-south vs east-west distinction still holds.

### ⚠️ Things they probe
- **North-south (ingress) vs east-west (mesh)** — the one-line differentiator.
- A mesh's headline features: **mTLS + traffic management + observability via sidecars**, no app changes.
- **You run both**: ingress at the edge, mesh inside. They're not alternatives.
- Meshes cost complexity/resources — **don't add one without a reason**.

---

### 🧪 Real-World Example

> "External traffic came through the **NGINX ingress controller** (host/path routing, TLS). Internally we had ~40 microservices, so we ran **Linkerd** for **automatic mTLS** and per-call golden-signal metrics, plus retries/timeouts we didn't want to reimplement in every service. Ingress was the front door; the mesh governed everything once traffic was inside."

---

### Key takeaway

> "**Ingress controller = north-south (external → cluster) L7 routing + TLS at the edge.** **Service mesh = east-west (service↔service) traffic via sidecars, adding mTLS, retries/timeouts, traffic splitting, and per-call observability with no app changes.** They're **complementary** — use both — but only add a mesh when many services justify its overhead (Linkerd if you want it lighter than Istio)."

### Related
- [Ingress vs LoadBalancer](./21-lb-vs-ingress.md) · [Ingress controller vs Ingress](./23-ingress-controller-vs-ingress.md)
- [How containers communicate / services](./04-pod-ip-communication.md) · [Network policies](./12-network-policies.md)
- [API Gateway vs Ingress & JWT](./39-api-gateway-vs-ingress-and-jwt.md)
