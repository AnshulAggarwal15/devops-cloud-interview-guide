## API Gateway vs Ingress, and JWT Authentication at the Gateway

### Question  
What's the difference between an **API Gateway** and an **Ingress**? (Not just AWS API Gateway — open-source in-cluster gateways too.) And for a concrete case: you want the gateway to do **JWT authentication** against an **external authorization server** — what do you configure to make that work?

### Short explanation of the question  
Two parts: (1) **Ingress = L7 routing into the cluster**, while an **API Gateway adds API-management concerns** (auth, rate limiting, transformation, quotas). (2) For **JWT auth**, the gateway must fetch the auth server's **public keys (JWKS)** to **verify the token signature** and validate claims (`iss`, `aud`, `exp`) — not just check a password.

---

### Answer  
- **Ingress** routes external HTTP(S) traffic to Services by **host/path** and terminates TLS — that's mostly it (via an ingress controller like NGINX).
- An **API Gateway** sits at the edge of your APIs and adds **authentication/authorization, rate limiting, quotas, request/response transformation, API keys, versioning, and observability**. It can be **cloud** (AWS API Gateway) or **in-cluster open source** (**Kong, APISIX, Ambassador/Emissary, Gloo**, or the **Kubernetes Gateway API**).
- For **JWT auth to an external IdP**, configure the gateway with the **issuer (`iss`)**, the **JWKS URI** (to fetch signing keys and verify the signature), the expected **audience (`aud`)**, allowed **algorithms**, the **token source** (`Authorization: Bearer`), and **claim validation/mapping**. The gateway then validates every request's token before proxying.

---

### Detailed explanation of the answer for readers’ understanding

### 🔀 Ingress vs API Gateway
| | Ingress | API Gateway |
|---|---|---|
| Primary job | **L7 routing** (host/path) into the cluster | **API management** at the edge |
| Auth | Basic/none (annotations for simple auth) | **JWT/OAuth2/OIDC, API keys, mTLS** |
| Traffic control | Minimal | **Rate limiting, quotas, throttling** |
| Transformation | No | **Request/response rewrite, header/body transform** |
| Examples | NGINX Ingress, AWS ALB Controller | **Kong, APISIX, Ambassador, Gloo**, AWS API Gateway |
| Overlap | — | Many API gateways *also* do ingress routing (Gateway API blurs the line) |

**One-liner:** *Ingress gets traffic to the right service; an API Gateway governs how the API is consumed (authn/z, limits, transforms).* For simple internal routing, Ingress is enough; when you expose APIs to partners/clients, you want a gateway.

### 🔐 JWT authentication at the gateway (the scenario)
A JWT is a **signed** token from an authorization server. "Authenticated" means the gateway **cryptographically verifies** it — so the essential piece is the **public keys**, fetched from the IdP's **JWKS endpoint**.

**What you configure on the gateway (JWT plugin/filter):**
| Setting | Why it's needed |
|---|---|
| **Issuer (`iss`)** | The trusted auth server URL; reject tokens from any other issuer |
| **JWKS URI** (or discovery `/.well-known/openid-configuration`) | Where the gateway fetches the IdP's **public signing keys** to verify the signature (and refreshes on key rotation) |
| **Audience (`aud`)** | Ensure the token was minted **for this API**, not another |
| **Allowed algorithms** | e.g. `RS256`; reject `none`/weak algs |
| **Token source** | Usually the **`Authorization: Bearer <jwt>`** header |
| **Claim validation** | `exp`/`nbf` (expiry), plus scopes/roles for authorization |
| **Claim → header mapping** | Forward `sub`/roles to the upstream service as headers |
| **Clock skew / caching** | Tolerate small time drift; cache JWKS to avoid hammering the IdP |

**Request flow:**
```text
Client ──(Authorization: Bearer JWT)──► API Gateway
  Gateway: fetch JWKS from IdP (cached) → verify signature
           check iss / aud / exp / scopes
     valid → forward request (+ mapped claim headers) to Service
   invalid → 401 Unauthorized (never reaches the backend)
```

**Two token models to distinguish:**
- **Local JWT validation** (most common, fast): gateway validates the signature offline using JWKS. No call to the IdP per request.
- **Token introspection / OAuth2** (opaque tokens): gateway calls the IdP's **introspection endpoint** to validate each token — simpler revocation, more latency.

### ⚠️ Things they probe
- The must-have for JWT is the **JWKS/public key** to **verify the signature** — not a shared password (that would be basic auth).
- Validate **`iss` + `aud` + `exp`**, restrict the **algorithm**, and handle **key rotation** (re-fetch JWKS).
- Name a concrete open-source gateway (**Kong/APISIX**) and where the JWT plugin sits.

---

### 🧪 Real-World Example

> “We fronted our APIs with **Kong** in the cluster. The **JWT/OIDC plugin** was configured with our auth server's **issuer** and **JWKS URI**, expected **audience**, and `RS256`. Kong verified the `Authorization: Bearer` token's signature against the cached JWKS, checked `iss/aud/exp` and scopes, then forwarded `sub` and roles as headers to the upstream. Invalid tokens got a 401 at the edge, so backends never saw unauthenticated traffic.”

---

### Key takeaway

> “**Ingress = L7 routing; API Gateway = API management (authn/z, rate limits, transforms)** — cloud or open-source (Kong/APISIX/Ambassador). For **JWT auth to an external IdP**, the gateway needs the **issuer, JWKS URI (to verify the signature), audience, allowed algorithms, token source, and claim validation** — verifying `iss/aud/exp` on every request and 401-ing invalid tokens before they reach the backend.”

### Related
- [Ingress vs LoadBalancer](./21-lb-vs-ingress.md) · [Ingress controller vs Ingress](./23-ingress-controller-vs-ingress.md)
- [GitHub/OIDC federation (JWT + JWKS in another context)](../10-aws/26-github-actions-oidc-federation.md)
