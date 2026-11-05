# Amagi — DevOps / Platform Engineer Interview

**Round style:** long, conversational, and adaptive — many questions were **follow-ups triggered by an incomplete answer**. Several would not have been asked at all if the earlier answer had been complete, so the notes below flag "what they were really driving at."

**Themes:** platform deployment under restricted access + cross-ecosystem auth (OIDC/IRSA), Kubernetes DNS internals (CoreDNS), pod IAM/IMDS, multi-cluster observability, Argo CD/GitOps, and API gateways.

> How to read this: each item has an **"As asked"** (cleaned but faithful to the messy phrasing), **"What they're really asking"**, a short answer, and a link to the full write-up.

---

## Section 1 — Platform deployment, restricted access & cross-ecosystem auth

### Q1 — Version control & error tracking when deploys go through a controlled service
**As asked (paraphrased):**
> "Debug actions run in a way that's in opposition to the customer being able to run the workflow — the feature where you can manually copy code and run it goes through a deployment service. Under such **restrictions**, how do you do **version controlling**? How do you go back and check **error tracking** — figure out which **release** created the problem? How can all this be accommodated?"

**What they're really asking:** when you can't freely touch the runtime and everything ships through a deployment service, how do you keep **traceability** — version every release and correlate errors back to the exact release?

**Short answer:**
- **Immutable, versioned artifacts**: every build → an image tagged with the **git SHA / semver**, pushed to a registry. The Git commit is the source of truth (**GitOps**).
- **Release traceability**: the deployment service records **which version is deployed where**; stamp the running workload with the version (labels/env), so logs/metrics/traces carry a `version`/`release` label.
- **Error tracking by release**: in observability, **group errors by `version` label** (and use an error tracker like Sentry with a release tag). A spike lines up with the release that introduced it.
- **Rollback**: revert the Git commit / pin the previous image tag — the deployment service redeploys the known-good version.

**Backed by:** [Argo CD, GitOps & image updates](../05-cicd/16-argocd-gitops-and-image-updates.md) · [Multi-cluster observability](../09-observability/12-multi-cluster-central-observability.md)

### Q2 — Cross-ecosystem access: "a role is correct, but a role alone won't work"
**As asked (paraphrased):**
> "For EC2 → S3 you'd use an **IAM role / instance profile** — that's within the AWS ecosystem. But **Databricks** (or GitHub) is a **separate ecosystem**. An IAM role is correct, but *what accompanies the role* is what matters. Even in our own AWS account, how do you authorize an external system with a suitable role? Role alone won't solve it — how does this integration work? We don't have access from our VCS to their AWS account, but we do have access within our cluster."

**What they're really asking:** an external (non-AWS) identity can't use an instance profile. What makes a role assumable by an outside system? **Identity federation** (OIDC/SAML) + a **scoped trust policy** + **STS `AssumeRoleWithWebIdentity`**.

**Short answer:**
- A role needs a **trust policy** that trusts an **external OIDC/SAML identity provider** — that's "what accompanies the role."
- Register the provider's **OIDC identity provider** in IAM, scope the role's trust policy to a specific **subject** (repo/branch, or workload), and the external system exchanges a **signed JWT** for **temporary STS credentials**. **No static keys.**

**Backed by:** [GitHub Actions OIDC → AWS federation](../10-aws/26-github-actions-oidc-federation.md)

### Q3 — How GitHub Actions ↔ AWS OIDC works; what you configure on AWS; how it's validated
**As asked (paraphrased):**
> "As a platform-engineering practice — the role is correct, but a role alone won't work because **GitHub is external**. How does this authentication work? How does GitHub Actions get an **OIDC** token, and how does the OIDC integration between **GitHub Action, GitHub, and the AWS account** work? Or explain **IRSA** if you know that. What do you configure on the **AWS side**? And **how does it validate** it?"

**Short answer:**
- GitHub mints a **signed OIDC JWT** for the workflow (claims: `iss`, `aud=sts.amazonaws.com`, `sub=repo:org/repo:ref`).
- **AWS side:** an **IAM OIDC identity provider** for `token.actions.githubusercontent.com` + an **IAM role** whose trust policy allows `sts:AssumeRoleWithWebIdentity` **only** for your scoped `sub`.
- **Validation:** STS verifies the **JWT signature against GitHub's JWKS** and checks `iss`/`aud`/`sub` against the trust policy → returns temporary creds.
- **IRSA** is the same mechanism with the **EKS cluster as the OIDC provider** and a **ServiceAccount** as the subject.

**Backed by:** [GitHub Actions OIDC → AWS federation](../10-aws/26-github-actions-oidc-federation.md) · [IRSA](../10-aws/104-irsa.md)

---

## Section 2 — Kubernetes DNS internals (CoreDNS)

### Q4 — How does Service A resolve Service B? How does A know where CoreDNS is? Who updates that config?
**As asked (paraphrased):**
> "Service A resolves to Service B — how does it happen? A must know **where CoreDNS is** — how does that happen? When you create even a test pod (busybox/ubuntu), **somebody has to update** 'here's my CoreDNS IP, use it for name resolution.' How? **kube-proxy, right?**"

**What they're really asking:** the full resolution path and, specifically, **who writes `/etc/resolv.conf`** (it's the **kubelet**, not kube-proxy — the leading hint was wrong).

**Short answer:**
- A resolves `service-b.ns.svc.cluster.local` via **CoreDNS** → gets Service B's **ClusterIP**; **kube-proxy** (iptables/IPVS) then routes that ClusterIP to a healthy pod.
- A knows CoreDNS because its **`/etc/resolv.conf`** has `nameserver <kube-dns ClusterIP>`.
- **The kubelet writes `resolv.conf`** into each pod (from `--cluster-dns` + the pod's `dnsPolicy`). **Not kube-proxy** — kube-proxy only programs the ClusterIP routing.

**Backed by:** [CoreDNS deep dive](../08-kubernetes/38-coredns-deep-dive-and-troubleshooting.md)

### Q5 — Why does a successful `nslookup google.com` still show ~3 errors?
**As asked (paraphrased):**
> "If you query **google.com** and CoreDNS is healthy, will you get a response with **no error at all**? It's forwarded to an external authoritative name server and you get a response — a successful query — but are there errors behind it? Can you tell me **why exactly three errors**? Your go-to was `nslookup`, but…"

**What they're really asking:** the **`ndots:5` + search-domain** behavior — a single external lookup fires several failed cluster lookups first.

**Short answer:** Yes, there are ~**3 NXDOMAIN errors** behind the success. With `options ndots:5` and a 3-entry search list, `google.com` (1 dot < 5) is tried as `google.com.<ns>.svc.cluster.local`, `google.com.svc.cluster.local`, `google.com.cluster.local` (**3 failures**) before `google.com.` succeeds. Fix/optimize with **FQDNs (trailing dot)**, tuned `ndots`, and **NodeLocal DNSCache**.

**Backed by:** [CoreDNS deep dive — the `ndots:5` section](../08-kubernetes/38-coredns-deep-dive-and-troubleshooting.md)

### Q6 — CoreDNS crashing intermittently with 6k–10k pods
**As asked (paraphrased):**
> "One CoreDNS instance, ~6,000–10,000 pods, and CoreDNS keeps crashing **very intermittently**. How do you troubleshoot? (You mentioned query rate, error rate, latency, cache hit.) What do you mean by **optimizing DNS / reducing error rates**? Why do error rates occur during a rolling deployment?"

**Short answer:**
- Find the restart cause: `logs --previous`/`describe` → usually **OOMKilled / CPU throttle / conntrack exhaustion**. Bump **resources + replicas**, autoscale CoreDNS with the **cluster-proportional-autoscaler**.
- Biggest win at scale: **NodeLocal DNSCache** (per-node cache) → far fewer queries to central CoreDNS, lower latency, less conntrack pressure.
- **Optimizing** = higher cache hit, fewer upstream forwards, fewer search-domain lookups (`ndots`/FQDN).
- **Rolling-deploy error spikes**: endpoints churn — clients/caches hit terminating or not-yet-ready pods → transient NXDOMAIN/SERVFAIL until things converge.

**Backed by:** [CoreDNS deep dive — troubleshooting at scale](../08-kubernetes/38-coredns-deep-dive-and-troubleshooting.md)

---

## Section 3 — Pod IAM / IMDS

### Q7 — Pod has no IRSA and should assume the node role, but it's not working
**As asked (paraphrased):**
> "A pod with **no IRSA** is expected to **assume the node role**. It's not working, though the pod is on that node and everything looks OK. **Where do you start?** (Could it be an **IMDS** issue — the IMDSv2 token, or a network config problem?)"

**Short answer:** Debug the **credential chain** then the **IMDS path**. The classic cause is the **IMDSv2 hop limit = 1** — a pod needs **2 hops** to reach `169.254.169.254`, so raise `httpPutResponseHopLimit` to **2**. Also check IMDS isn't disabled/blocked, no stray SA `role-arn` annotation is forcing IRSA, and the node role's permissions/trust are correct. Best practice: **give the pod its own IRSA role** instead of the node role.

**Backed by:** [Pod node-role / IMDS troubleshooting](../10-aws/27-pod-node-role-imds-troubleshooting.md)

---

## Section 4 — Multi-cluster observability

### Q8 — Central observability for 20–30 clusters
**As asked (paraphrased):**
> "20–30 clusters. I won't install Grafana on all of them — I want **central observability**: one Grafana that sees every pod/node/CoreDNS across all clusters. Lay out a simple architecture. Where's Prometheus — central or per cluster? What does the **central cluster contain**? What's **Grafana's data source**? Does Grafana talk to all 20 Prometheis, or does something central consolidate them?"

**Short answer:**
- **Per cluster:** a **Prometheus agent** scrapes locally and **`remote_write`s** out (no Grafana per cluster).
- **Central cluster:** a consolidation layer — **Thanos / Mimir / Cortex** on **S3** — that ingests from all clusters.
- **Grafana:** one instance, **single data source = the central query layer** (Thanos Query / Mimir), **not** the 20 Prometheis.

**Backed by:** [Multi-cluster central observability](../09-observability/12-multi-cluster-central-observability.md)

### Q9 — How do you know which metric came from which cluster?
**As asked (paraphrased):**
> "All 20 clusters emit node/pod/CoreDNS/kube-proxy metrics. How do you make sure 'this is from cluster A, this is B, this is C'?"

**Short answer:** Each Prometheus agent stamps unique **`external_labels: { cluster: "a", region: "..." }`**, so every series carries a `cluster` label you filter/group by in Grafana (and Thanos/Mimir use it for dedup). The label set must be **unique per cluster** or series collide.

**Backed by:** [Multi-cluster observability — `external_labels`](../09-observability/12-multi-cluster-central-observability.md)

### Q10 — ELK retention and "what is the size of the Elasticsearch disk?"
**As asked:** "What's the data retention in the ELK stack — 30 days? **What is the size of the Elasticsearch disk?**"

**What they're really asking:** do you understand your own logging infra's **sizing**, and can you reason about it?

**Short answer (how to reason, since it's your infra):** ES disk ≈ **daily ingest (GB/day) × retention (days) × (1 + replica count) × (1 − compression), plus ~15–20% headroom**. So quote it as, e.g., "~X GB/day × 30 days × 2 (1 replica) ≈ Y, plus headroom." Levers to mention: **retention window, replica count, index lifecycle (hot-warm-cold), and compression**. Know your real numbers if it's your system; otherwise show the formula and the levers.

---

## Section 5 — GitOps / Argo CD

### Q11 — What do you do with Argo CD, and how does it deploy a new ECR image?
**As asked (paraphrased):**
> "You use Argo CD daily — but **how is it integrated, with what**? What do you generally **do** with it? You deploy resources through Argo CD — walk me through that **pipeline**. You push a new image to **ECR**; if there's a new version, Argo CD picks it up and deploys? **How does Argo CD know** there's a 3.5 in ECR when 3.4 is running?"

**What they're really asking (the trap):** Argo CD is **GitOps** — it watches **Git, not the registry**. So a new ECR image does **nothing** until its **tag is updated in Git**.

**Short answer:**
- Argo CD reconciles the cluster to a **Git** repo of manifests (Helm/Kustomize) — sync status, auto-sync, self-heal, rollback via Git.
- Pipeline: **CI builds → pushes image to ECR → CI updates the image tag in the Git manifests repo → Argo CD sees the commit → syncs**.
- "How does it know about 3.5?" → **because the tag in Git changed** (CI committed it), **or** you run **Argo CD Image Updater**, which polls ECR and writes the new tag back to Git. Argo CD does **not** watch ECR by default.

**Backed by:** [Argo CD, GitOps & image updates](../05-cicd/16-argocd-gitops-and-image-updates.md)

---

## Section 6 — API Gateway

### Q12 — API Gateway vs Ingress, and JWT auth to an external auth server
**As asked (paraphrased):**
> "Difference between **API Gateway and Ingress**? (Not AWS API Gateway — there are open-source ones, and you can run an API gateway at the Kubernetes level.) Use case: set up **JWT-based authentication** (not basic auth) against an **external authorization server**. What components do you configure in the API gateway so authentication succeeds?"

**Short answer:**
- **Ingress = L7 routing** into the cluster; **API Gateway = API management** (authn/z, rate limiting, transforms) — cloud or open-source (**Kong, APISIX, Ambassador, Gloo**).
- **JWT auth** config: the **issuer (`iss`)**, the **JWKS URI** (to fetch the auth server's public keys and **verify the token signature**), the expected **audience (`aud`)**, allowed **algorithms** (e.g. RS256), the **token source** (`Authorization: Bearer`), and **claim validation** (`exp`, scopes) with optional claim→header mapping. Invalid tokens get **401 at the edge**.

**Backed by:** [API Gateway vs Ingress & JWT](../08-kubernetes/39-api-gateway-vs-ingress-and-jwt.md)

---

## How to handle this style of interview (your meta-question)
This round adapted to your answers — several follow-ups appeared only because an earlier answer was thin. Two things fix that:
1. **Answer in the shape they want**: state the **direct answer first**, then the **mechanism**, then a **real example**. Vague first answers invite the drill-down.
2. **Know the "why" one layer deeper** for your daily tools. You use Argo CD, CoreDNS, IRSA every day — the questions were all "how does it actually work underneath?" The linked notes each end with a **one-liner** built for exactly that.

## One-line recap to rehearse
- **External auth:** "Role alone can't be assumed by an outside system — you need **OIDC/SAML federation + a scoped trust policy**, then `AssumeRoleWithWebIdentity` for temp creds. IRSA is the same with EKS as the provider."
- **DNS path:** "**kubelet writes resolv.conf**, CoreDNS resolves name→ClusterIP, **kube-proxy** routes ClusterIP→pod."
- **ndots:** "`ndots:5` makes `google.com` try 3 search domains first → **3 NXDOMAIN errors** before success."
- **CoreDNS at scale:** "usually **OOM/conntrack** — more replicas+resources, proportional autoscaler, **NodeLocal DNSCache**."
- **Pod IAM:** "no IRSA → node role via IMDS; the classic fix is **IMDSv2 hop limit = 2**."
- **Central observability:** "**agent + remote_write → Thanos/Mimir → one Grafana**; tell clusters apart with **`external_labels`**."
- **Argo CD:** "watches **Git, not ECR** — new image deploys when its **tag is committed to Git** (CI or Image Updater)."
- **Gateway JWT:** "gateway needs **JWKS + iss + aud + alg** to verify the token signature and claims."

---

# Amagi — Round 2 (Networking, HTTP, TLS & Terraform drift)

**Round style:** a long **hands-on + networking deep-dive**. It opened with **debugging a pod without any observability stack**, moved through **IRSA trust internals** and **pod→internet networking**, then a **latency scenario** (users in India hitting a **US-East** API) drilled all the way down the stack — decompose latency (DNS/TCP/TLS/TTFB) → keep-alive → HTTP versions → multiplexing → TLS versions → the TLS handshake — and ended on **Terraform drift**.

> ⚠️ This transcript had the transcribing **LLM answering inline**, and the "You said:" blocks mix the **interviewer's questions** with the **candidate's own (nervous, breaking-English) attempts**. The LLM answers were **mostly accurate** (HTTP/2-3, TLS, multiplexing, drift); the weak spots were the **candidate's own articulation** — flagged below. All topics are now dedicated notes.

## Section 7 — Get a pod's logs with NO observability stack (and no Argo CD)
**As asked (paraphrased):**
> "Assume no observability setup and no Argo CD access. **How will you get the logs of that running pod?** Once you `exec` inside, can you see the pod's logs there? And **where do the pod logs actually come from** — who's sending them?"

**What they're really asking:** can you debug with plain `kubectl`, and do you understand that **`kubectl logs` (stdout/stderr via the API) ≠ going inside with `exec`**?

**Short answer:** Use **`kubectl logs <pod>`** (add **`--previous`** for the earlier crash, `-c` for a container, `-f` to stream) — no tooling needed. The app writes to **stdout/stderr** → the **container runtime** captures it → the **kubelet** exposes it → **`kubectl logs` reads it via the API**. From **inside** (`exec`) you **can't** get those container logs (no file if it logs to stdout, and `kubectl` isn't installed in the pod) — you can only read app **log files** if the app writes them; otherwise exit and use `kubectl logs`.

**Backed by:** [Getting pod logs without observability](../08-kubernetes/48-getting-pod-logs-without-observability.md)

## Section 8 — IRSA: how does AWS actually trust/identify the pod?
**As asked (paraphrased):**
> "You told me how the role attaches to the ServiceAccount and how the workload uses it — but **how exactly is the trust established between the pod and AWS**? The OIDC token — **how is it generated**? To put the pod in the trust relationship you need **something that identifies the pod**. What is it?"

**What they're really asking:** what **identity claim** lets AWS trust a specific pod — not an IP or pod name, but the **projected OIDC token's issuer + namespace + ServiceAccount + audience** matched against the role's trust policy.

> **[Candidate struggled to articulate this]** The content exists in the round-1 IRSA note; the gap was **saying it crisply**.

**Short answer:** The pod doesn't "trust AWS." EKS **projects a signed OIDC JWT** (for the pod's ServiceAccount) into the pod. On `AssumeRoleWithWebIdentity`, **STS validates that token against the cluster's EKS OIDC provider** and checks its claims — **issuer + namespace + ServiceAccount + audience** — against the **IAM role's trust policy**. Match → temporary credentials. So the pod is identified by **`namespace:serviceaccount` in the projected token**, never by IP/pod name.

**Backed by:** [IRSA](../10-aws/104-irsa.md) · [GitHub/OIDC federation](../10-aws/26-github-actions-oidc-federation.md) · [AuthN vs AuthZ](../10-aws/29-authentication-vs-authorization.md)

## Section 9 — Pod → google.com: how does the request travel?
**As asked (paraphrased):**
> "You said you're a bit less strong on networking — basic question: from a pod you hit `google.com`. **How exactly does the request travel** from the pod to Google?"

**Short answer:** Two stages. **DNS:** the pod asks **CoreDNS**, which **forwards** `google.com` to the upstream resolver → IP. **Data path:** the **VPC CNI** gave the pod a real VPC IP → out the **node ENI** → the subnet **route table** sends `0.0.0.0/0` to a **NAT Gateway** (private subnet) → **Internet Gateway** → internet, and back the same way. On failure check **DNS → route table/NAT/IGW → SG → NACL (stateless) → NetworkPolicy**.

**Backed by:** [Pod → internet egress path](../08-kubernetes/49-pod-to-internet-egress-path.md) · [CoreDNS deep dive](../08-kubernetes/38-coredns-deep-dive-and-troubleshooting.md)

## Section 10 — NLB vs target: where is the time being spent?
**As asked (paraphrased):**
> "The NLB and targets are in the same region. If something's taking longer, **how will you find out where** — the LB path or the target?"

**Short answer:** Compare **NLB CloudWatch metrics** (target response/processing latency, connections, resets) against **app/pod metrics** and **access logs** with request start/end timestamps. If the LB path is slow but the app is fast → network/LB path; if the request reaches the target fast but the app takes seconds → **pod CPU/mem, workers/threads, DB, downstream APIs**. The question to answer: *"is the time getting to the target, or processing after it arrives?"*

**Backed by:** [Diagnosing latency with curl timing](../04-networking/14-diagnosing-latency-curl-timing.md) · [Load balancers & 502s](../10-aws/106-load-balancers-and-502.md)

## Section 11 — Decompose the latency: DNS / TCP / TLS / TTFB
**As asked (paraphrased):**
> "You're in Bangalore, the load balancer is in US-East. It could be **TCP/TLS handshake**, **TTFB**, or the **application**. **How will you find out which** is the exact problem — is it TLS/TCP, or TTFB?"

**Short answer:** Break the request into phases with **`curl -w`**: **DNS (`time_namelookup`) / TCP (`time_connect − namelookup`) / TLS (`time_appconnect − connect`) / TTFB (`time_starttransfer − appconnect`)**. If **setup (DNS/TCP/TLS)** dominates → network distance/handshake cost. If those are normal but **TTFB** is high → the backend path — and **TTFB isn't pure app time**, so split **NLB target latency** from **app/DB** before blaming the app.

**Backed by:** [Diagnosing latency with curl timing](../04-networking/14-diagnosing-latency-curl-timing.md)

## Section 12 — The latency scenario & connection reuse
**As asked (paraphrased):**
> "Keep-alive only helps users **reusing** a connection — new users still hit delays. If new users keep arriving, keep-alive won't help. **What do you do to resolve it?**"

**What they're really asking:** do you know that keep-alive can't help connections that don't exist yet, and can you reach for the **real fixes** — reduce handshake cost *and* move the endpoint closer?

**Short answer:** New clients still pay **TCP + TLS setup** at the full India→US-East RTT. Reduce it with **HTTP/2-3 multiplexing, TLS 1.3 + session resumption, connection pooling**; but if it's **physical distance**, the real fix is **edge TLS termination (CDN) or a closer region** with latency routing. Protocol tuning cuts *handshake overhead*, not the base RTT.

**Backed by:** [Cross-region latency & connection reuse](../04-networking/13-latency-cross-region-and-connection-reuse.md)

## Section 13 — What is multiplexing?
**As asked:** "What exactly do you mean by multiplexing requests?"

**Short answer:** Multiple independent requests/responses sharing **one** TCP/TLS connection **concurrently** (HTTP/2 splits them into **streams/frames**), so you don't pay a new handshake per request. (LLM answer here was **correct**.)

**Backed by:** [HTTP/1.1 vs 2 vs 3 & multiplexing](../04-networking/11-http-1-2-3-and-multiplexing.md)

## Section 14 — HTTP/1.1 vs HTTP/2 vs HTTP/3
**As asked:** "Do you know HTTP 1.1, 2, 3? What's the difference?"

**Short answer:** **1.1 = TCP, ~one request per connection at a time; 2 = TCP + multiplexing + HPACK (but TCP-level HOL blocking remains); 3 = QUIC over UDP + TLS 1.3 integrated, independent streams (no transport HOL blocking), faster setup + connection migration.** (LLM comparison table was **correct**.)

**Backed by:** [HTTP/1.1 vs 2 vs 3 & multiplexing](../04-networking/11-http-1-2-3-and-multiplexing.md)

## Section 15 — TLS versions (1.2 vs 1.3) & the handshake
**As asked (paraphrased):**
> "TLS 1.1/1.2/1.3 differences? When was 1.3 rolled out, and what were 1.2's limitations? **What happens in a TLS handshake?**" Then the candidate tried to describe the handshake ("server supports some key algorithm, client sends the respective keys, then some assumption…").

**What they're really asking:** can you walk the handshake accurately — and do you know the **private key is never sent**?

> **[Candidate's own answer was shaky here]** The transcript shows groping through the handshake and implying keys are exchanged directly. The correction (which the LLM gave): **private keys never travel**; the client verifies the **public certificate**, and both sides **derive** the same symmetric session key via the key exchange.

**Short answer:** Handshake = **negotiate → authenticate (server certificate) → key exchange → derive symmetric session keys → encrypted data**. **TLS 1.3** = fewer round trips (1-RTT, 0-RTT on resume), removes legacy crypto, mandates forward secrecy; keep **1.2 + 1.3** enabled (disable 1.0/1.1). On Nginx: `ssl_protocols TLSv1.2 TLSv1.3;`.

**Backed by:** [TLS handshake & versions](../04-networking/12-tls-handshake-and-versions.md) — this is the one to over-prepare, since it's where the round got wobbly.

## Section 16 — Terraform drift (during `plan`)
**As asked (paraphrased):**
> "You run `terraform plan` / `pulumi up` and find **drift**. What do you do? Any possibility to **import** resources? Would you use `terraform state` commands?"

**Short answer:** Decide **which side is the source of truth**: if the manual change was **intended**, **codify it** in `.tf` and apply; if **not intended**, **re-`apply`** to reconcile reality back to config. Use **`terraform import`** for a real resource **not yet in state** (not for hiding drift). **Don't** use `state rm`/manual state edits as the normal drift fix — that hides the problem. (LLM answer + candidate's instinct here were **broadly correct**; the candidate correctly noted forcing infra to match state can be destructive.)

**Backed by:** [Terraform drift](../06-terraform/17-terraform-drift.md) · [Apply fails halfway](../06-terraform/16-apply-fails-halfway.md) · [Import blocks & state recovery](../06-terraform/103-import-blocks-and-state-recovery.md)

---

## Round 2 — how you did & what to tighten
- **Good instincts:** you knew `kubectl logs`/`--previous` for pod debugging, correctly reasoned keep-alive doesn't help *new* connections, and your Terraform-drift answer (forcing infra to match state can be destructive) was sound.
- **Where you wobbled (be honest with yourself):**
  - **IRSA trust articulation** — you understood the pieces but couldn't cleanly say *"the pod is identified by `namespace:serviceaccount` in a projected OIDC token that STS validates against the trust policy."* Drill note 104 until that one sentence is automatic.
  - **TLS handshake mechanics** — you groped for the steps and implied keys are exchanged directly. Lock in: **negotiate → authenticate (cert) → key exchange → derive session keys → encrypt**, and **"the private key never leaves the server."** Note 12.
  - **Networking** — you told the interviewer you're "a bit less on networking," and it showed on pod→internet. Notes 49 (egress path) and 14 (curl decomposition) are the highest-leverage fixes.
- **Pattern (same as round 1):** you *recognize* topics ("yes, I read about them") but get caught on the **exact mechanism / crisp phrasing** under pressure. Recognition ≠ recall. The nervous, breaking-English stretches in the transcript line up exactly with the topics above — that's nerves compounding a recall gap, and both improve with **saying the one-liners out loud** until they're reflex.

## Round 2 — one-line recaps to rehearse
- **Pod logs (no tooling):** "`kubectl logs --previous`; app→stdout→runtime→kubelet→API. `exec` gets you *inside*, not the stdout logs."
- **IRSA trust:** "The pod is identified by **`namespace:serviceaccount` in a projected OIDC token**; STS validates it against the cluster's OIDC provider + the role's trust policy — no IP/pod name."
- **Pod→internet:** "CoreDNS forwards the name upstream; then **CNI IP → node ENI → route table → NAT → IGW**."
- **Latency decomposition:** "`curl -w`: DNS / TCP / TLS / TTFB — setup-heavy = distance/handshake; TTFB-heavy = backend (and TTFB isn't pure app time)."
- **Latency fix:** "Keep-alive only helps reused connections; new clients still handshake at full RTT. Cut overhead with HTTP/2-3 + TLS 1.3 + pooling, but fix distance with **edge termination (CDN) or a closer region**."
- **Multiplexing:** "Many requests over **one** connection concurrently via streams/frames."
- **HTTP:** "1.1 TCP serial-ish; 2 TCP multiplexed; 3 QUIC/UDP, no transport HOL blocking, TLS 1.3 built in."
- **TLS:** "Negotiate → authenticate (cert) → key exchange → derive session keys → encrypt. **Private key never sent.** 1.3 = fewer round trips + forward secrecy."
- **Drift:** "Decide the source of truth — codify intended changes, re-apply to revert unintended ones, `import` unmanaged resources; don't hide drift with state edits."
