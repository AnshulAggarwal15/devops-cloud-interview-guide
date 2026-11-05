# Kubernetes — One-Shot Handbook

> Read this top to bottom. It follows **one app** from "I have a container" to "running reliably in
> production," introducing each concept exactly when you'd actually hit it. Every section notes **why it
> matters** and **📊 whether it's used in industry**. The other files in this folder (`01`–`29`) are
> deep-dives on individual pieces; this file is the map that connects them.

## Table of contents

1. You start with a container
2. The cluster: control plane + worker nodes
3. You declare desired state (the core mental model)
4. Reconciliation: how it self-heals
5. The Pod
6. The Deployment
7. Scheduling
8. Networking problem → the Service
9. Service types
10. Exposing HTTP → Ingress
11. Config & secrets → ConfigMap / Secret
12. State & storage → Volumes, PVC, StatefulSet
13. Health checks → probes
14. Scaling → HPA + node autoscaling
15. Multi-tenancy & security → Namespaces, RBAC, ServiceAccounts, NetworkPolicy
16. Packaging & deploying → Helm, GitOps (Argo CD)
17. Observability
18. The whole thing in one paragraph
19. Quick-reference cheat table

---

## 1. You start with a container

You've built your app into a **Docker image** (blueprint) and can run it as a **container** (running instance). On one laptop, `docker run` is fine.

**The problem that creates Kubernetes:** in production you need *many* copies across *many* machines, automatic restart when one dies, rolling upgrades with no downtime, networking between them, secrets, and scaling under load. Doing that by hand is impossible. **Kubernetes is a container orchestrator** that does all of it for you.

📊 *Containers are universal; Kubernetes is the de-facto orchestrator for anything non-trivial.*

## 2. The cluster: control plane + worker nodes

- **Control plane (the brain):**
  - **API server** — the single front door; everything talks to it (you, controllers, nodes).
  - **etcd** — the database; stores the *entire desired + current state* of the cluster.
  - **Scheduler** — decides *which node* a new Pod runs on.
  - **Controller manager** — runs the reconciliation loops (§4).
- **Worker nodes (the muscle):** each runs
  - **kubelet** — node agent; ensures the Pods assigned here are actually running.
  - **kube-proxy** — programs the networking rules for Services (§8).
  - **container runtime** (containerd) — actually runs the containers.

📊 *Almost nobody self-manages the control plane. You use **EKS/GKE/AKS** — the cloud runs the control plane; you manage worker nodes (or go serverless with Fargate / Karpenter).*

## 3. You declare desired state (the mental model that explains everything)

You never tell Kubernetes "start this container." You **declare the end state** in YAML ("I want 3 replicas of this image") and submit it via `kubectl` → **API server** → stored in **etcd**. This **declarative model** is the single most important idea in Kubernetes.

📊 *Universal — and it's exactly why GitOps works (§16).*

## 4. Reconciliation: how it self-heals

For every resource, a **controller** runs a loop: *compare desired state (etcd) vs actual state → make actual match desired.* Declared 3 replicas but one Pod crashed? The controller sees 2 ≠ 3 and creates a new one. **This loop is why Kubernetes self-heals** — no human paged at 3am for a dead Pod.

📊 *The beating heart of K8s; also the model behind Operators and Argo CD.*

## 5. The Pod — smallest unit (but you rarely create it directly)

A **Pod** wraps one (sometimes a few tightly-coupled) containers that share a network namespace and IP. It's the smallest thing Kubernetes schedules.

**Critical property: Pods are ephemeral and disposable.** They die, get rescheduled, and come back with a **new IP**. You almost never create bare Pods — a bare Pod that dies is just gone. You need a controller to manage them.

📊 *Create Pods directly only for quick debugging. Real workloads always go through a controller.*

## 6. The Deployment — how you actually run apps

A **Deployment** manages Pods for you (via a **ReplicaSet** under the hood). You declare "3 replicas of image v2," and it:

- keeps exactly 3 running (self-healing),
- does **rolling updates** (replace Pods gradually → zero downtime),
- supports **rollback** (`kubectl rollout undo`).

```yaml
apiVersion: apps/v1
kind: Deployment
metadata: { name: orders }
spec:
  replicas: 3
  selector: { matchLabels: { app: orders } }   # which pods it manages
  template:
    metadata: { labels: { app: orders } }        # labels stamped on each pod
    spec:
      containers:
        - name: orders
          image: myrepo/orders:v2
```

**Labels** (`app: orders`) are arbitrary metadata used to *select/group* Pods. They're the glue - everything uses (Deployments, Services, NetworkPolicies) — but by themselves they change nothing.

📊 ***Deployment is the default** for stateless apps — the single most-used workload object.*

## 7. Scheduling — which node?

When Pods are created, the **scheduler** places them based on: resource **requests/limits** (CPU/memory), **taints & tolerations** (a node repels Pods unless they tolerate it — e.g. reserve GPU nodes), and **affinity/anti-affinity** (prefer/avoid certain nodes, or spread across zones).

📊 *requests/limits and multi-AZ spreading are everywhere; taints/affinity used for GPU pools, spot instances, isolation.*

## 8. Networking problem → the Service

Your 3 Pods each have an IP, but those IPs **change constantly**. How does anything reach them reliably? The **Service**.

A **Service** is a **stable virtual IP + DNS name** in front of a changing set of Pods (selected by labels), with **built-in load balancing**. The key insight: it's **not a proxy box** — it's realized by:

- **EndpointSlices** — the live list of *healthy* Pod IPs behind it, and
- **kube-proxy** — iptables/IPVS rules on **every node** that rewrite Service-IP traffic to a real Pod IP.

**Service discovery:** **CoreDNS** lets Pods reach a Service by name — `http://orders` resolves to the Service's ClusterIP. Service-to-service calls use names, never IPs.

```yaml
apiVersion: v1
kind: Service
metadata: { name: orders }
spec:
  selector: { app: orders }   # picks all pods labelled app=orders
  ports:
    - port: 80                 # the Service's port
      targetPort: 8080         # the container's port
  type: ClusterIP
```

📊 *Services + CoreDNS are in literally every cluster.*

## 9. Service types (how far the exposure reaches)

- **ClusterIP** (default) — internal only. Service-to-service. *(most common)*
- **NodePort** — also opens a static port (30000–32767) on **every node's IP**. Rarely used alone (ugly ports); it's the building block under LoadBalancer. Here "service" = the K8s Service object; your app is reached *through* it.
- **LoadBalancer** — asks an installed provider/controller for an external frontend for the Service. It may be public or private and may use a Layer 4 or Layer 7 product depending on the implementation. Backends may be node ports or Pod IPs; do not assume it always means an NLB.
- **Headless** (`clusterIP: None`) — no virtual IP; DNS returns individual endpoint addresses. Common with StatefulSets and other clients that need direct endpoint discovery.

📊 *ClusterIP is the usual internal default; external exposure depends on controller, policy, and public/private configuration.*

## 10. Exposing HTTP the smart way → Ingress

If every externally exposed service had its own load balancer, cost and configuration would grow quickly. **Ingress** resources define HTTP(S) host/path routing such as "host `app.com` + path `/orders` → the `orders` Service," often with centralized TLS.

But **Ingress is only the API configuration**—an **Ingress Controller** (for example ingress-nginx or AWS Load Balancer Controller) must implement it. The concrete path varies. Two common shapes are:

```text
Client → DNS → Cloud LB → in-cluster Ingress controller → Service → Pod
Client → DNS → controller-managed L7 load balancer → Pod IP or node target
```

**L7 vs L4 nuance:** Ingress rules describe HTTP(S), Layer 7 behavior. A `LoadBalancer` Service exposes declared TCP/UDP/SCTP ports, but the cloud product created and its capabilities are controller/provider decisions—not universally "L4/NLB." The newer **Gateway API** provides a more expressive, role-oriented successor for many use cases.

## 11. Config & secrets → ConfigMap and Secret

Don't bake config into images. **ConfigMap** holds non-sensitive config; **Secret** holds sensitive data (base64-encoded, ideally encrypted at rest). Inject them as **env vars** or **mounted files**.

**Classic gotcha:** if you mount config as **env vars**, changing the ConfigMap does **not** update running Pods — env is set once at container start, so you must **restart/roll the Deployment**. If mounted as a **volume**, the file updates eventually, but your app must *re-read* it.

📊 *ConfigMaps everywhere; Secrets often backed by external stores — **Vault**, AWS Secrets Manager, External Secrets Operator - because native Secrets are only base64, not encrypted by default.*

## 12. State & storage → Volumes, PVC, StatefulSet

Containers lose their filesystem on restart. For persistence: a **Volume** → backed by a **PersistentVolume
(PV)** → claimed via a **PersistentVolumeClaim (PVC)** → provisioned dynamically by a **StorageClass** (e.g.
an EBS volume). Apps needing stable identity + storage (databases, Kafka) use a **StatefulSet** instead of a
Deployment.

📊 *For databases most teams use **managed services (RDS/DynamoDB)** rather than in-cluster state — but
StatefulSets are common for Kafka / Elasticsearch / Prometheus.*

## 13. Health checks → probes

- **Liveness probe** — "is it alive?" Fails → **restart** the container. (Fixes hangs/deadlocks.)
- **Readiness probe** — "can it serve traffic yet?" Fails → **removed from the Service's endpoints** (no restart). This is how rolling updates avoid sending traffic to not-yet-ready Pods.
- **Startup probe** — for slow-booting apps, so liveness doesn't kill them prematurely.

📊 *liveness + readiness are effectively mandatory in any serious deployment.*

## 14. Scaling → HPA + node autoscaling

- **Horizontal Pod Autoscaler (HPA)** — adds/removes *Pods* based on CPU/memory/custom metrics.
- **Cluster Autoscaler / Karpenter** — adds/removes *nodes* when Pods can't be scheduled (no capacity).

📊 *HPA is standard; **Karpenter** is now the popular choice on AWS for fast, flexible node scaling.*

## 15. Multi-tenancy & security → Namespaces, RBAC, ServiceAccounts, NetworkPolicy

- **Namespaces** — logical partitions (team-a vs team-b, dev vs prod-in-cluster).
- **RBAC** — who can do what (users/roles → verbs on resources).
- **ServiceAccount** — the identity a Pod uses to talk to the API — and, via **IRSA** on EKS, to assume an AWS IAM role for cloud permissions (no static keys in Pods).
- **NetworkPolicy** — *optional* firewall between Pods. By default the network is **flat/open**; you add policies to *restrict*. Enforced only if your **CNI** (Calico/Cilium) supports them. Labels don't require policies — policies *use* labels to target Pods.

📊 *Namespaces + RBAC everywhere; **IRSA is a must-know for EKS**; NetworkPolicies common in
security-conscious orgs, absent in many others.*

## 16. Packaging & deploying → Helm, then GitOps (Argo CD)

Raw YAML per environment doesn't scale.

- **Helm** — the K8s package manager; a **chart** = templated manifests + a `values.yaml`, so you deploy the
same app to dev/stage/prod with different values.
- **Kustomize** — template-free alternative; patches/overlays base YAML per environment.
- **GitOps (Argo CD)** — **Git is the single source of truth.** Argo CD *watches* your Git repo of
manifests/Helm charts and **continuously reconciles** the cluster to match it — with **self-heal** (reverts
manual drift) and **OutOfSync** detection. Deploy = commit; roll back = revert the commit.

📊 *Helm is standard packaging; **Argo CD + GitOps is the modern deployment norm** — and it's explicitly in
the Docusign JD.*

## 17. Observability

Three pillars: **metrics** (Prometheus), **logs** (ELK/Loki), **traces** (Jaeger/OpenTelemetry) — visualized
in **Grafana**.

📊 *Prometheus + Grafana is the default stack.*

---

## 18. The whole thing in one paragraph (say this if asked "explain Kubernetes")

> "You declare desired state to the **API server**, stored in **etcd**. **Controllers** constantly reconcile
> actual toward desired, which is why it self-heals. You run apps via **Deployments**, which keep N **Pods**
> alive and perform controlled rolling updates. Since Pods are ephemeral with changing IPs, a **Service** gives
> a stable virtual IP and load distribution using a Service data plane such as **kube-proxy** plus
> **EndpointSlices**; **CoreDNS** provides name-based discovery. External HTTP can be described by an
> **Ingress** or **Gateway API** resource and implemented by its controller. A `LoadBalancer` Service asks a
> provider/controller for an external frontend whose concrete Layer 4/Layer 7 product and target mode are
> implementation-dependent. Config comes from **ConfigMaps/Secrets**, persistence from **PVCs**, health from
> probes, scaling from **HPA plus node autoscaling**, security from **RBAC/workload identity/NetworkPolicy**,
> and manifests are commonly packaged with **Helm/Kustomize** and deployed through **GitOps**."

---

## 19. Quick-reference cheat table


| Concept                 | One-liner                                              | Layer/Type               | Industry use                  |
| ----------------------- | ------------------------------------------------------ | ------------------------ | ----------------------------- |
| Pod                     | Smallest schedulable unit; wraps containers            | —                        | Usually controller-managed    |
| Deployment              | Manages interchangeable Pods and rolling updates       | Workload                 | Default for stateless apps    |
| ReplicaSet              | Maintains replica count under a Deployment             | Workload                 | Managed by Deployment         |
| StatefulSet             | Stable identity; optional per-Pod PVC templates        | Workload                 | Kafka/ES/Prometheus and peers |
| Service (ClusterIP)     | Stable virtual IP and distribution over endpoints      | Service abstraction      | Everywhere internally         |
| NodePort                | Allocated port exposed on eligible node addresses      | Service exposure         | Usually indirect              |
| LoadBalancer Service    | Requests an external provider/controller frontend      | Implementation-dependent | Edge/private integration      |
| Headless Service        | DNS returns endpoint addresses; no virtual IP          | Service discovery        | Stateful/direct discovery     |
| Ingress + controller    | HTTP(S) host/path routing implemented by a controller  | Layer 7 API              | Common for HTTP               |
| ConfigMap / Secret      | Inject non-secret/sensitive configuration              | —                        | Everywhere                    |
| PV / PVC / StorageClass | Persistent storage request/provisioning model          | Storage                  | Stateful workloads            |
| Probes                  | Startup, traffic readiness, and safe liveness checks   | Health                   | Common; must be designed well |
| HPA                     | Autoscale Pods using resource/custom metrics           | Autoscaling              | Standard                      |
| Karpenter / CA          | Add/remove nodes for pending/capacity needs            | Autoscaling              | Common                        |
| Namespace / RBAC        | Object scope and Kubernetes API authorization          | Security                 | Everywhere                    |
| ServiceAccount / IRSA   | Workload identity for API/cloud access                 | Security                 | Must-know on EKS              |
| NetworkPolicy           | Directional, additive allow-lists when enforced        | Network security         | Security-conscious orgs       |
| Helm / Kustomize        | Package/compose Kubernetes manifests                   | Delivery                 | Standard                      |
| Argo CD / GitOps        | Reconcile cluster state from version-controlled config | Delivery                 | Common modern pattern         |


---

## 20. Is this enough for mid-level or senior interviews?

This handbook is a **map and first-pass explanation**, not proof of operational depth.

- It is enough to begin **mid-level questions** and identify weak areas.
- It is enough to begin **senior scenario questions as a diagnostic**.
- It is **not enough by itself** to claim mid-senior Kubernetes depth or reliably pass a senior troubleshooting round.

Before treating Kubernetes as an interview strength, you should be able to do these without reading the answer:

```text
[ ] Explain kubectl apply through authn/authz/admission, etcd, controllers,
    scheduling, kubelet/runtime, CNI/CSI, readiness, and EndpointSlices
[ ] Deploy an app with requests/limits, probes, configuration, identity,
    Service, ingress/Gateway, policy, topology, PDB, and autoscaling
[ ] Diagnose Running-but-no-traffic, Pending, CrashLoopBackOff,
    ImagePullBackOff, OOMKilled, DNS failure, blocked policy, and Pending PVC
[ ] Explain CPU throttling, OOM kills, QoS, node pressure, eviction, drain,
    graceful termination, readiness, and PDB limitations
[ ] Explain CSI provisioning, access/reclaim modes, AZ topology, snapshots,
    backup, and a tested restore
[ ] Plan an upgrade around version skew, deprecated APIs, add-ons, node drains,
    rollback, etcd backup, and control-plane recovery
[ ] Design least-privilege RBAC/workload identity, admission policy, secret
    handling, and image/supply-chain controls
[ ] Define useful SLIs/SLOs, dashboards, alerts, capacity signals, and an
    incident response for the workload
```

Use senior questions now, but use this loop:

```text
Answer closed-book → expose the gap → open the matching 01–29 deep dive
→ reproduce it in a cluster → break it → diagnose it → answer again
```

A correct verbal answer is the start. Mid-senior evidence is being able to investigate with commands, interpret results, choose a mitigation, explain trade-offs, and prevent recurrence.

---

*This file is the map. For depth on any single box, open the matching `01`–`29` file in this folder.*