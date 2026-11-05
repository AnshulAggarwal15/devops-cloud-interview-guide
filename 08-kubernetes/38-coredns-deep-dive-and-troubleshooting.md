## CoreDNS Deep Dive — Resolution Path, `ndots`, and Troubleshooting at Scale

### Question  
How does **Service A resolve Service B**? How does a pod even know where CoreDNS is, and who configures that? Why does a successful `nslookup google.com` still produce **errors** in CoreDNS metrics? And how do you troubleshoot CoreDNS **crashing intermittently** in a cluster with thousands of pods?

### Short explanation of the question  
This goes well past "CoreDNS does service discovery." It tests the **full resolution path** (`/etc/resolv.conf` → CoreDNS → kube-proxy), **who writes what** (kubelet vs kube-proxy — a common mix-up), the **`ndots:5` search-domain behaviour** that generates failed lookups, and **scaling/crash troubleshooting** for CoreDNS under heavy load.

---

### Answer  
- **Service A → Service B**: A resolves `service-b.ns.svc.cluster.local` via **CoreDNS**, which returns Service B's **ClusterIP**; then **kube-proxy** (iptables/IPVS) load-balances that ClusterIP to a healthy **endpoint (pod IP)**.
- **How A knows where CoreDNS is**: the pod's **`/etc/resolv.conf`** has `nameserver <kube-dns ClusterIP>` (a stable, well-known IP like `10.96.0.10`).
- **Who writes it**: the **kubelet** injects `resolv.conf` into every pod based on the pod's **`dnsPolicy`** and the kubelet's `--cluster-dns` setting — **not kube-proxy**. (kube-proxy only programs the ClusterIP→pod routing.)
- **The "errors" behind a successful lookup**: because of **`ndots:5` + the search list**, resolving `google.com` first tries several cluster search domains → **~3 NXDOMAIN errors** → then the absolute name succeeds. The lookup "works" but the metrics show errors.
- **Crashing at scale**: CoreDNS is usually **OOMKilled / CPU-starved / conntrack-exhausted** under load — fix with **more replicas + resources, autoscaling, caching, and NodeLocal DNSCache**.

---

### Detailed explanation of the answer for readers’ understanding

### 🔗 The full resolution path (Service A → Service B)
```text
Pod A app
  │ 1. looks up "service-b.ns.svc.cluster.local"
  │    (reads nameserver from /etc/resolv.conf → kube-dns ClusterIP)
  ▼
CoreDNS (kube-system)
  │ 2. watches the API server; returns Service B's ClusterIP (an A record)
  ▼
Pod A now has ClusterIP → opens connection
  │ 3. kube-proxy's iptables/IPVS rules DNAT the ClusterIP
  ▼
A healthy endpoint (Pod B's IP) from Service B's EndpointSlice
```
Two different components: **CoreDNS resolves the name → ClusterIP**; **kube-proxy routes the ClusterIP → a pod**.

### 🧭 How the pod knows the CoreDNS IP — and who writes it
- Every pod's **`/etc/resolv.conf`** looks roughly like:
```text
nameserver 10.96.0.10
search ns.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```
- The **kubelet** writes this file when it creates the pod, using its configured **cluster DNS IP** (`--cluster-dns`) and the pod's **`dnsPolicy`** (default **`ClusterFirst`**). So when you spin up a busybox/ubuntu test pod, **nobody manually tells it the CoreDNS IP — the kubelet injects it**.
- Common interview trap: "kube-proxy does it, right?" → **No.** kube-proxy programs the ClusterIP routing (so `10.96.0.10` reaches CoreDNS pods); the **kubelet** is what populates `resolv.conf`. The kube-dns **Service** having a stable ClusterIP is what makes `10.96.0.10` a fixed target.

### 🔢 Why a successful `nslookup google.com` shows ~3 errors (`ndots:5`)
- `options ndots:5` means: if a name has **fewer than 5 dots**, the resolver tries it **with each search domain first**, only falling back to the name **as-is** at the end.
- `google.com` has **1 dot** (< 5), so with a 3-entry search list the resolver queries:
```text
1. google.com.ns.svc.cluster.local   → NXDOMAIN (error)
2. google.com.svc.cluster.local      → NXDOMAIN (error)
3. google.com.cluster.local          → NXDOMAIN (error)
4. google.com.                       → SUCCESS (forwarded upstream)
```
- So a **single** external lookup generates **3 failed (NXDOMAIN) queries** before the 4th succeeds. `nslookup` reports success, but CoreDNS metrics rightly count **3 errors** behind it. That's the answer the interviewer was fishing for.
- **Fixes / optimizations**: use **FQDNs with a trailing dot** (`google.com.`) to skip the search list; lower `ndots` for specific pods via `dnsConfig`; add **NodeLocal DNSCache**; and cache negative responses. This "search-domain amplification" is also why external-heavy workloads hammer CoreDNS.

### 📈 CoreDNS metrics to watch
**Query rate, error rate (SERVFAIL/NXDOMAIN), latency (p99), cache hit ratio**, plus pod **CPU/memory** and restarts. "Optimizing DNS / reducing error rate" = higher cache hit, fewer upstream forwards, fewer search-domain lookups, enough replicas/resources.

### 💥 Troubleshooting CoreDNS crashing intermittently at 6k–10k pods
Work it as: **symptoms → resource/scale → query pattern → node/network**.
1. **Why is it restarting?** `kubectl describe`/`logs --previous` → almost always **OOMKilled** (memory too low for the cache/load) or CPU throttled. Bump **requests/limits** and **replica count**.
2. **Scale CoreDNS to the fleet**: a couple of replicas can't serve 10k pods. Use the **cluster-proportional-autoscaler** to scale CoreDNS with node/core count.
3. **Cut the load with NodeLocal DNSCache**: a per-node DNS cache (DaemonSet) answers most queries locally, slashing traffic to central CoreDNS and reducing tail latency and conntrack pressure — the single biggest win at this scale.
4. **conntrack / UDP issues**: massive DNS (UDP :53) can **exhaust conntrack** on nodes and trigger the classic race → intermittent timeouts. Raise `nf_conntrack_max`, consider `use-vc`/TCP, and NodeLocal DNSCache mitigates this too.
5. **Search-domain amplification**: `ndots:5` multiplies external lookups ~4x → use FQDNs / tune `ndots` / cache negatives.
6. **Upstream health**: if `forward . /etc/resolv.conf` upstream is flaky, you get **SERVFAIL** spikes — check the upstream resolver and add caching.

### ⚠️ Why error rates rise during a rolling deployment
During a rollout, **endpoints churn**: pods terminate while clients/caches still hold their records, and new pods aren't `Ready` yet. Lookups to just-removed endpoints or not-yet-ready services produce transient **NXDOMAIN/SERVFAIL** until endpoints and caches converge. Readiness probes, sane TTLs, and NodeLocal DNSCache reduce the blip.

---

### 🧪 Real-World Example

> “Our CoreDNS pods OOMKilled intermittently at ~8k pods. `logs --previous` showed OOM; we raised memory, scaled replicas with the **cluster-proportional-autoscaler**, and rolled out **NodeLocal DNSCache**. DNS latency and error rate dropped sharply. We also found external lookups were 4x-amplified by `ndots:5`, so we switched hot external names to FQDNs.”

---

### Key takeaway

> “Resolution: **resolv.conf (written by the kubelet) → CoreDNS resolves name→ClusterIP → kube-proxy routes ClusterIP→pod.** A successful `google.com` lookup still logs **~3 NXDOMAIN errors** because **`ndots:5`** tries the search domains first. At scale, CoreDNS crashes are usually **OOM/CPU/conntrack** — fix with **more replicas+resources, proportional autoscaling, caching, and NodeLocal DNSCache**.”

### Related
- [Role of CoreDNS](./17-core-dns.md) (basics)
- [Pod-to-pod / service communication](./04-pod-ip-communication.md) · [Service & kube-proxy](./08-service-relates-to-kubeproxy.md)
- [Metrics to scrape with Prometheus](../09-observability/03-metrics-to-scrape-with-prometheus.md)
