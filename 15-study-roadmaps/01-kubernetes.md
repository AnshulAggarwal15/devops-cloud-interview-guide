# Kubernetes Study Roadmap — Two Paths

Built from your **honest self-assessment** (spoken answers, no LLM). Baseline verdict: **~4.5/10 vs a mid-senior bar — junior with a few early-mid patches.** Real conceptual bits exist (declarative model, control plane, Deployment/ReplicaSet, ConfigMap/Secret, rolling update). The gap is **keywords without mechanisms**, and it's disqualifying for the depth you're being interviewed at.

## Your specific gaps (both paths target these)
Ranked by how badly they hurt you in real rounds:
1. **Identity/auth mechanism** — RBAC objects, ServiceAccount vs user, **IRSA/OIDC trust chain** (drilled in ≥3 interviews; you had keywords only).
2. **Networking** — Service **types** (you missed ClusterIP, miscalled CoreDNS/Ingress "types"), **CoreDNS resolution path**, kube-proxy, **pod→internet egress**.
3. **Architecture holes** — you **forgot the kubelet** and mislabeled CoreDNS as a node component.
4. **Workloads** — **StatefulSet & DaemonSet: zero.**
5. **Scheduling/resources** — **requests/limits behavior** (limits → OOMKill/throttle, not "alerts"), **cordon/drain**, startup probe + ordering.
6. **Scaling** — **HPA vs Cluster Autoscaler** (pods vs nodes), KEDA.
7. **Troubleshooting** — CrashLoop/Pending **mechanisms** (not "403"), Pending ≠ probe failure.
8. **Delivery** — **Helm** (it's a package manager), **Argo CD reconciliation** (you use it daily but couldn't say what it does).

---

## The one rule that makes either path work: **active recall, out loud**
You have ~50 good notes and a 4.5/10 — because **reading ≠ knowing.** Both paths below use the same discipline:
- **Learn** (read the note) → **Do** (a hands-on lab, or at minimum say it) → **Prove** (close the note and *say the mechanism out loud in your own words*; if you ramble or fall back to keywords, you don't know it yet — repeat tomorrow).
- Treat every note's **"Key takeaway" one-liner** as the target you must be able to reproduce unaided.
- If you can't **teach it to an imaginary junior in 60 seconds**, it's not learned.

---

# PATH 1 — The real path (8–12 weeks, ~10–15 hrs/week) → mid-senior competence + CKA

This is the one I want you to succeed on. It's **hands-on first** — you cannot learn Kubernetes by reading, and your assessment is proof. End state: you can *operate and explain* a cluster, and CKA is the forcing function.

### Phase 0 — Environment & kubectl fluency (Week 1)
- **Do:** install **kind** (or minikube) locally; later spin a small **EKS** cluster with `eksctl`. Deploy your first Deployment + Service. Live in `kubectl`.
- **Master:** `get / describe / logs / logs --previous / exec / apply / rollout / explain / get events`.
- **Prove:** deploy nginx, break it, read the events, fix it — no notes.

### Phase 1 — Architecture & the request lifecycle (Week 1–2)  *(fixes gap #3)*
- **Learn:** [cluster architecture](../08-kubernetes/01-cluster-architecture.md), [components interaction](../08-kubernetes/02-components-interaction.md), [pod scheduling lifecycle](../08-kubernetes/44-pod-scheduling-lifecycle.md), [one-shot handbook](../08-kubernetes/00-kubernetes-one-shot-handbook.md).
- **Do:** `kubectl get pods -A -n kube-system` — find the API server, etcd, scheduler, controller-manager, **and the kubelet on each node**. Watch a pod get scheduled (`--watch` + `describe`).
- **Prove:** narrate `kubectl apply` → API server (authN/authZ/admission) → etcd → controllers → **scheduler (filter/score/bind)** → **kubelet** → CRI/CNI → Ready. Out loud. (You forgot the kubelet last time — this is the fix.)

### Phase 2 — Workloads (Week 2–3)  *(fixes gap #4)*
- **Learn:** [Deployment/StatefulSet/DaemonSet](../08-kubernetes/42-deployment-statefulset-daemonset.md), [deployment strategies](../08-kubernetes/16-all-deployment-strategies.md).
- **Do:** deploy a **Deployment** (scale/roll it), a **StatefulSet** (see stable `-0/-1` names + per-pod PVC), a **DaemonSet** (one pod per node). Delete a StatefulSet pod and watch it come back with the *same* identity/volume.
- **Prove:** "Deployment = stateless replicas via ReplicaSet; StatefulSet = stable identity + per-pod storage + ordered; DaemonSet = one per node." + one real use of each.

### Phase 3 — Networking (Week 3–5, spend extra here)  *(fixes gap #2 — your weakest)*
- **Learn:** [types of services](../08-kubernetes/05-types-of-services.md), [NodePort vs LB](../08-kubernetes/07-nodeport-vs-lb.md), [headless service](../08-kubernetes/10-headless-service.md), [kube-proxy](../08-kubernetes/08-service-relates-to-kubeproxy.md), [CoreDNS deep dive](../08-kubernetes/38-coredns-deep-dive-and-troubleshooting.md), [pod/service comms](../08-kubernetes/04-pod-ip-communication.md), [ingress vs LB](../08-kubernetes/21-lb-vs-ingress.md), [ingress controller vs ingress](../08-kubernetes/23-ingress-controller-vs-ingress.md), [pod→internet egress](../08-kubernetes/49-pod-to-internet-egress-path.md), [service mesh vs ingress](../08-kubernetes/45-service-mesh-vs-ingress.md).
- **Do:** create a **ClusterIP**, **NodePort**, and **LoadBalancer** service and see the difference. `kubectl exec` into a pod → `cat /etc/resolv.conf`, `nslookup <svc>`, `nslookup google.com`. Install an ingress controller and route two hosts.
- **Prove:** list the **real** service types (ClusterIP/NodePort/LoadBalancer/ExternalName — ClusterIP is default); walk **Service A → CoreDNS → ClusterIP → kube-proxy → pod**; walk **pod → CoreDNS → CNI → node ENI → route table → NAT → IGW → internet.**

### Phase 4 — Config, secrets & identity (Week 5–6, spend extra here)  *(fixes gap #1 — most-drilled)*
- **Learn:** [secret types](../08-kubernetes/35-types-of-secrets.md), [RBAC](../08-kubernetes/36-rbac.md), [IRSA](../10-aws/104-irsa.md), [authN vs authZ framework](../10-aws/29-authentication-vs-authorization.md), [GitHub/OIDC federation](../10-aws/26-github-actions-oidc-federation.md), [pod node-role/IMDS](../10-aws/27-pod-node-role-imds-troubleshooting.md).
- **Do:** create a Role + RoleBinding for a read-only ServiceAccount; test with `kubectl auth can-i --as=system:serviceaccount:...`. On EKS, wire **IRSA**: OIDC provider → IAM role trust policy scoped to `namespace:serviceaccount` → annotate SA → watch a pod call S3 with no keys.
- **Prove (say all three unaided):**
  - RBAC objects: **Role/RoleBinding (namespaced), ClusterRole/ClusterRoleBinding (cluster-wide).**
  - **ServiceAccount = workload identity, not a human user.**
  - **IRSA trust chain:** "the pod's projected **OIDC JWT** (claims: issuer + `namespace:serviceaccount` + audience) is validated by **STS** against the cluster's OIDC provider and the role's trust policy → temporary creds." Authentication = the token; authorization = the role's policy.

### Phase 5 — Scheduling & resources (Week 6–7)  *(fixes gap #5)*
- **Learn:** [requests & limits](../08-kubernetes/31-requests-and-limits.md), [QoS](../08-kubernetes/30-kubernetes-qos.md), [liveness/readiness/startup](../08-kubernetes/20-liveness-vs-readiness.md), [taints & tolerations](../08-kubernetes/18-taints-tolerations.md), [cordon/drain vs taint](../08-kubernetes/47-cordon-drain-vs-taint.md), [node affinity](../08-kubernetes/27-node-affinity.md).
- **Do:** set a low memory **limit** and watch the pod get **OOMKilled**; set a CPU limit and watch **throttling**. Add a **taint**, then a toleration. **cordon** then **drain** a node and watch pods move.
- **Prove:** "requests = scheduling guarantee; limit exceeded → **CPU throttle / memory OOMKill** (not alerts)." "startup probe runs **first** and disables liveness/readiness until it passes." "cordon = no new pods; drain = cordon + evict; taint = repel unless tolerated."

### Phase 6 — Scaling (Week 7–8)  *(fixes gap #6)*
- **Learn:** [horizontal vs vertical](../08-kubernetes/34-horizontal-vs-vertical-scaling.md), [advanced autoscaling: custom metrics + KEDA](../08-kubernetes/37-advanced-autoscaling-custom-metrics-keda.md).
- **Do:** create an **HPA**, generate load, watch pods scale. On EKS, watch the **Cluster Autoscaler/Karpenter** add a node when pods go Pending.
- **Prove:** "**HPA scales pods; Cluster Autoscaler scales nodes.** HPA defaults to CPU/mem; custom metrics via the **Prometheus Adapter**; event-driven via **KEDA** (which manages its own HPA)."

### Phase 7 — Troubleshooting (Week 8–9)  *(fixes gap #7)*
- **Learn:** [CrashLoopBackOff](../08-kubernetes/19-crashloopbackoff.md), [Pending/ImagePull/node-down](../08-kubernetes/43-troubleshooting-pending-imagepull-node-down.md), [logs without observability](../08-kubernetes/48-getting-pod-logs-without-observability.md), [debug-order playbook](../docusign-debug-order-playbook.md).
- **Do:** deliberately cause each — bad command (CrashLoop), huge resource request (Pending), wrong image tag (ImagePull), `drain` a node — and debug each with `describe` + `logs --previous`.
- **Prove:** "CrashLoop = container starts then **crashes repeatedly** (bad config/missing dep/OOM) → `logs --previous`. **Pending = can't schedule** (capacity/taints/affinity/PVC), *not* probes. ImagePull = image name/tag/registry auth."

### Phase 8 — Delivery & operations (Week 9–10)  *(fixes gap #8)*
- **Learn:** [Helm essentials](../08-kubernetes/46-helm-essentials.md), [rollback strategy](../08-kubernetes/14-rollback-strategy.md), [Argo CD/GitOps](../05-cicd/16-argocd-gitops-and-image-updates.md), [CI/CD from scratch](../05-cicd/17-cicd-from-scratch-gitops-deep-dive.md).
- **Do:** package your app as a **Helm chart** (values + a `range`); install/upgrade/rollback. Install **Argo CD**, point it at a Git repo, change the repo, watch it **reconcile**; then `git revert` and watch it roll back.
- **Prove:** "Helm = **package manager**: chart (templates + values) → release. Argo CD = **GitOps**: reconciles the cluster to the **desired state in Git**, self-heals drift; rollback = revert the commit."

### Phase 9 — CKA prep + mock drilling (Week 10–12)
- **Do:** a CKA course (KodeKloud) + **killer.sh** timed sessions. Then drill the interview banks **out loud**: [common AWS/SRE](../14-previous-interview-questions/05-common-aws-devops-sre-questions.md), [20 questions](../14-previous-interview-questions/06-twenty-real-interview-questions.md), [K8s/TF/Helm bank](../14-previous-interview-questions/07-k8s-terraform-helm-aws-docker-bank.md).
- **Prove:** take the **CKA**. It will expose anything still shaky, and it's the credential that changes how interviewers treat you.

**Weekly cadence:** ~3 sessions of 3–4 hrs. Each session: 60% hands-on, 25% notes, 15% out-loud recall of prior weeks (spaced repetition). Re-drill an earlier phase's one-liners every week so they don't fade.

---

# PATH 2 — The crash path (roughly 5–10 days, heavy daily hours) → survive the deep interview

> **Read this first — honest risk:** this path is **triage, not competence.** It can get you through a *talking* interview if you can articulate mechanisms crisply, because most of your losses were "keywords, no mechanism" — and that's fixable fast with drilling. **But:** if they make you **do** something live (whiteboard, `kubectl` troubleshooting, "show me"), or push one layer past what you rehearsed, it won't hold. And if it lands a role, you'll be under-water on day one. Use it only because the clock forces it — and keep doing Path 1 afterward regardless.

### The method (this *is* the path)
For each topic below: read the note's **Key takeaway**, then **say the mechanism out loud until it's automatic** (record yourself; if you ramble or drop to keywords, repeat). You're building **one crisp spoken answer + one layer of mechanism** per topic. No breadth, no labs beyond a half-day on kind to make it concrete.

### Priority order (by how hard it got drilled in your actual rounds)

**Day 1 — Identity/auth (your #1 killer, drilled every round)**
- [authN vs authZ framework](../10-aws/29-authentication-vs-authorization.md) — the backbone for every "how is it secure?" Learn to split **who (authN) / what (authZ)** and name a mechanism for each.
- [IRSA](../10-aws/104-irsa.md) + [RBAC](../08-kubernetes/36-rbac.md): drill the **IRSA trust-chain sentence** and the **4 RBAC objects** until reflexive.
- **Must be able to say:** the IRSA one-liner (projected OIDC JWT → STS validates issuer+`ns:sa`+aud against the trust policy → temp creds), RBAC objects, SA ≠ human user.

**Day 2 — Networking (your weakest domain)**
- [service types](../08-kubernetes/05-types-of-services.md) (get ClusterIP/NodePort/LoadBalancer/ExternalName **right** — ClusterIP is default), [CoreDNS resolution path](../08-kubernetes/38-coredns-deep-dive-and-troubleshooting.md), [kube-proxy](../08-kubernetes/08-service-relates-to-kubeproxy.md), [ingress vs LB](../08-kubernetes/21-lb-vs-ingress.md), [pod→internet egress](../08-kubernetes/49-pod-to-internet-egress-path.md).
- **Must be able to say:** Service A→B path; pod→internet path; ingress vs LB in one line.

**Day 3 — Architecture + workloads (stop the easy misses)**
- [cluster architecture](../08-kubernetes/01-cluster-architecture.md) + [scheduling lifecycle](../08-kubernetes/44-pod-scheduling-lifecycle.md): never forget the **kubelet**; CoreDNS is an **add-on**, not a node component.
- [Deployment/StatefulSet/DaemonSet](../08-kubernetes/42-deployment-statefulset-daemonset.md): the three-way distinction cold.
- **Must be able to say:** the `kubectl apply → Ready` flow; when to use StatefulSet vs DaemonSet.

**Day 4 — Troubleshooting (interview bread-and-butter)**
- [CrashLoop](../08-kubernetes/19-crashloopbackoff.md), [Pending/ImagePull/node-down](../08-kubernetes/43-troubleshooting-pending-imagepull-node-down.md), [logs without observability](../08-kubernetes/48-getting-pod-logs-without-observability.md).
- **Must be able to say:** what each state means + first debug move (`describe` → `logs --previous`). Pending = scheduling, not probes.

**Day 5 — Scheduling/resources + scaling + delivery (the rest of the frequent hits)**
- [requests/limits](../08-kubernetes/31-requests-and-limits.md) (OOM/throttle, **not** alerts), [probes + ordering](../08-kubernetes/20-liveness-vs-readiness.md), [cordon/drain vs taint](../08-kubernetes/47-cordon-drain-vs-taint.md).
- [HPA vs Cluster Autoscaler](../08-kubernetes/34-horizontal-vs-vertical-scaling.md) (pods vs nodes) + KEDA one-liner.
- [Helm](../08-kubernetes/46-helm-essentials.md) + [Argo CD](../05-cicd/16-argocd-gitops-and-image-updates.md): one crisp line each.

**Day 6 — HIGH-risk stretch (your interviewers actually went here — do NOT skip if you have the day)**
- **Security & hardening:** [cluster hardening](../08-kubernetes/40-cluster-hardening.md) + [NetworkPolicies](../08-kubernetes/12-network-policies.md) — CIS/kube-bench, Pod Security Standards, admission control (Kyverno/OPA), default-deny egress. *(Jio drilled this.)*
- **Advanced autoscaling:** [custom metrics + KEDA](../08-kubernetes/37-advanced-autoscaling-custom-metrics-keda.md) — Prometheus Adapter for custom metrics, KEDA multi-trigger (max wins), stabilization window for startup lag. *(Nielsen drilled this.)*
- **Service mesh vs ingress:** [45](../08-kubernetes/45-service-mesh-vs-ingress.md) — north-south (ingress) vs east-west (mesh: mTLS, retries, traffic-splitting via sidecars).
- **API Gateway vs Ingress + JWT:** [39](../08-kubernetes/39-api-gateway-vs-ingress-and-jwt.md) — gateway needs JWKS + iss + aud + alg to verify a token. *(Amagi drilled this.)*
- **Must be able to say (one crisp line each):** "hardening = CIS via kube-bench + PSS + admission control + default-deny NetworkPolicies"; "custom metrics via Prometheus Adapter, events via KEDA"; "mesh = east-west mTLS/traffic-mgmt via sidecars, ingress = north-south L7"; "gateway validates a JWT's signature via the IdP's JWKS + iss/aud."

**Day 6.5 — MEDIUM-risk fill-ins (fundamentals that still come up)**
- **Deployment strategies detail:** [16](../08-kubernetes/16-all-deployment-strategies.md) — rolling vs **blue-green** vs **canary**, and when to use each.
- **ConfigMap vs Secret:** [35](../08-kubernetes/35-types-of-secrets.md) — you scored well here, so just refresh the one-liner (config = non-sensitive; Secret = base64, not encrypted → external secrets/KMS).
- **Labels & selectors:** [06](../08-kubernetes/06-labels-selectors.md) — how Services/Deployments *find* their pods (the glue you skipped over in the drill).
- **Must be able to say:** "blue-green = two full envs, switch traffic; canary = shift a % and watch"; "a Service routes to pods whose labels match its selector via EndpointSlices."

**Days 7–10 (if you have them) — drill + a half-day on kind**
- Run the [K8s/TF/Helm question bank](../14-previous-interview-questions/07-k8s-terraform-helm-aws-docker-bank.md) and [20 questions](../14-previous-interview-questions/06-twenty-real-interview-questions.md) **out loud, timed.** Spin a **kind** cluster for a half day so you can honestly say "yes, I've done this" and survive a light live task.

> ⚠️ **Not covered even here (accepted gaps — and missing from the notes too):** **storage (PV/PVC/StorageClass/CSI)**, **Jobs/CronJobs**, **init/sidecar containers**, **namespaces/ResourceQuota/LimitRange**, and **etcd backup/restore + cluster upgrades**. If any of these come up, bridge honestly ("haven't gone deep there") rather than bluff. Storage is the most likely to be asked — worth a dedicated note (see below).

### Crash-path survival tactics (from your MoveInSync/Amagi rounds)
- **Answer shape:** direct answer → mechanism → (brief) example. Generic openers invite the drill-down that buried you.
- **Bridge honestly when stuck:** "I haven't run that myself, but conceptually it's X" — you did this well once; do it deliberately.
- **Lead with the authN/authZ split** on any security question. It's the single highest-leverage move.
- **Don't bluff a mechanism you don't have** — a confident wrong answer (like "CrashLoop = 403") is worse than "I'd check the events and logs first, then reason from what they show."

---

## Which one?
Do **Path 2 only if you have an imminent interview** you can't move. Otherwise **Path 1 is the answer**, and even on Path 2 you should roll into Path 1 the day the interview ends — because the crash path can win a screen but can't survive the job, and the next interview will drill just as deep.

## Progress tracker
Mark these as you can **say them unaided** (not "read them"):
- [ ] `kubectl apply → Ready` flow incl. kubelet
- [ ] Real service types + Service A→B path
- [ ] Pod → internet egress path
- [ ] IRSA trust chain (one sentence) + RBAC objects + SA≠user
- [ ] Deployment vs StatefulSet vs DaemonSet
- [ ] requests/limits behavior (OOM/throttle) + probe ordering
- [ ] cordon/drain vs taint
- [ ] HPA vs Cluster Autoscaler + KEDA
- [ ] CrashLoop / Pending / ImagePull meanings + first debug move
- [ ] Helm (package manager) + Argo CD (reconciles Git → cluster)
- [ ] Hardening (CIS/kube-bench/PSS/admission) + default-deny NetworkPolicies
- [ ] Advanced autoscaling (Prometheus Adapter custom metrics, KEDA, stabilization)
- [ ] Service mesh vs ingress (east-west vs north-south)
- [ ] API Gateway vs Ingress + JWT validation (JWKS/iss/aud)
- [ ] Deployment strategies: rolling vs blue-green vs canary
- [ ] Labels & selectors (how Services find pods)
