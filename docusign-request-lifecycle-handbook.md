<!-- #genai: senior-level end-to-end request lifecycle handbook — every component a request touches from client to pod to datastore and back, plus the deploy/IaC/observability context; one-stop recall for Docusign senior DevOps interview -->

# The Full Request Lifecycle — One-Stop Senior DevOps Recall

> Follow **one HTTPS API request** from a user's device, all the way into a pod on EKS, to the database, and
> back — naming **every component** on the path (big *and* small). Then the surrounding context: how the pod
> got there (CI/CD + GitOps), how the infra exists (Terraform), and how you'd see it all (observability).
>
> If you can narrate this diagram end to end, you can handle almost any "walk me through what happens
> when..." senior question.

---

## The master diagram

```text
                                    ┌──────────────────────── USER ────────────────────────┐
                                    │  Browser / mobile app  →  GET https://api.example.com  │
                                    └───────────────────────────┬───────────────────────────┘
                                                                │
        (1) DNS RESOLUTION                                      ▼
        Route 53 (hosted zone, A/ALIAS record) ──► returns IP of the edge / load balancer
        [TTL, health-checked failover, latency/geo routing]
                                                                │
        (2) EDGE / SECURITY                                     ▼
        CloudFront (CDN, caches static)  ─  AWS WAF (L7 rules: SQLi/XSS/rate-limit)  ─  Shield (DDoS)
                                                                │
        (3) TLS                                                 ▼
        TLS handshake terminated at the LB;  cert from ACM;  SNI, ALPN, HTTP/2
                                                                │
        (4) LOAD BALANCER (the ALB — sits in PUBLIC subnets, multi-AZ)                     
            ┌───────────────────────────────────────────────────────────────────────┐
            │  Listener (:443)  →  Rule (host/path match)  →  Target Group  ──────────┼──► targets
            │  [health checks decide which targets are eligible]                      │
            └───────────────────────────────────────────────────────────────────────┘
                                                                │  (target = the ingress-controller pods,
                                                                │   or nodes' NodePort)
        (5) VPC NETWORK PLUMBING (everything below is inside the VPC)                   
            Security Groups (stateful, instance/ENI)  ·  NACLs (stateless, subnet)
            Route tables  ·  public subnet (ALB/NAT)  ·  private subnets (nodes/pods/DB)
                                                                │
        ══════════════════════════ INTO THE EKS CLUSTER (private subnets) ══════════════════════════
                                                                │
        (6) INGRESS CONTROLLER  (nginx / AWS Load Balancer Controller) — pods running in the cluster
            reads Ingress objects (L7 host/path rules), may re-terminate TLS, adds headers
                                                                │
        (7) SERVICE (ClusterIP)  — stable virtual IP for the backend                    
            CoreDNS resolved the name → ClusterIP   |   kube-proxy (iptables/IPVS on each node)
            EndpointSlices = live list of healthy Pod IPs                                
                                                                │  (kube-proxy DNATs ClusterIP → a Pod IP)
        (8) THE NODE that owns the target Pod                                            
            kubelet (runs/watches pods)  ·  container runtime (containerd)  ·  CNI (VPC CNI: pod gets a real ENI IP)
            NetworkPolicy (Calico/Cilium) may allow/deny this hop
                                                                │
        (9) THE POD  (scheduled here earlier by the scheduler; kept alive by a Deployment/ReplicaSet)
            ┌──────────────────────────────────────────────────────────────────────┐
            │  initContainers (ran first)                                            │
            │  app container  ── readiness✓ (in endpoints) · liveness✓ · startup✓    │
            │  sidecar (Envoy for service mesh / log shipper)                        │
            │  resources: requests/limits · QoS class · could be OOMKilled           │
            │  config from: ConfigMap (env/volume) · Secret · downward API           │
            │  identity: ServiceAccount → IRSA → temp AWS creds                      │
            └──────────────────────────────────────────────────────────────────────┘
                                                                │
        (10) APP DOES ITS WORK — downstream calls (each has its own network + auth path)
             ├─ SQL DB: RDS/Aurora (private subnet, SG allows app-SG:5432, connection pool)
             ├─ Cache: ElastiCache/Redis (SG-gated) — read-through / write-through
             ├─ Object storage: S3 (via S3 Gateway Endpoint, IAM role, no internet)
             ├─ Queue/async: SQS / Kafka (decouple slow work to workers)
             ├─ Secrets: Secrets Manager / SSM / Vault (+ kms:Decrypt)
             └─ Other microservices: internal ALB / Service DNS / service mesh mTLS
                                                                │
        (11) RESPONSE TRAVELS BACK  ── Pod → Service → Ingress controller → ALB → (CloudFront) → client
             (stateful SG return traffic auto-allowed; NACL needs explicit ephemeral-port return)
```

---

## The numbered walk-through (say this; every noun is a talking point)

**(1) DNS.** The client resolves `api.example.com` via **Route 53** (hosted zone, A/ALIAS record). Route 53
can do **health-checked failover** and **latency/geo routing**; responses honor **TTL**.

**(2) Edge & security.** Request may hit **CloudFront** (CDN — serves cached static, terminates close to
user), screened by **AWS WAF** (L7 rules: SQL injection, XSS, **rate limiting**) and **Shield** (DDoS).

**(3) TLS.** The **TLS handshake** terminates at the edge/LB using a cert from **ACM**; **SNI** selects the
cert, **ALPN** negotiates **HTTP/2**.

**(4) Load balancer (ALB).** Lives in **public subnets across AZs**. Internals: a **Listener** (`:443`) →
**Rules** (host/path routing) → **Target Group** → **Targets**, with **health checks** gating eligibility.
(ALB = L7; an **NLB** would be L4 for raw TCP/static IPs.)

**(5) VPC plumbing.** Everything now is inside the **VPC**. Traffic is gated by **Security Groups**
(stateful, per-ENI) and optionally **NACLs** (stateless, per-subnet). **Route tables** define public
(→ IGW) vs private (→ NAT) subnets. ALB/NAT in **public** subnets; nodes, pods, DB in **private** subnets.

**(6) Ingress controller.** The ALB forwards to the **Ingress controller** pods (nginx or the **AWS Load
Balancer Controller**), which read **Ingress** objects and route at **L7** by host/path (may re-terminate
TLS, add/strip headers). *(The newer **Gateway API** is the emerging successor to Ingress.)*

**(7) Service.** The controller forwards to a **Service (ClusterIP)** — a **stable virtual IP** for the
backend. **CoreDNS** resolved the Service name → ClusterIP (client-side discovery). **kube-proxy**
(iptables/IPVS rules on every node) **DNATs** that virtual IP to a real Pod IP, chosen from the
**EndpointSlice** (only **ready** pods).

**(8) The node.** On the chosen node: **kubelet** (ensures pods run, executes probes), the **container
runtime** (**containerd**), and the **CNI plugin** (**AWS VPC CNI** gives each pod a real VPC ENI IP). A
**NetworkPolicy** (if Calico/Cilium) may permit/deny this hop.

**(9) The pod.** It was placed here earlier by the **scheduler** (respecting **requests/limits**,
**taints/tolerations**, **affinity/anti-affinity**, **topology spread**) and is kept at the desired count by
a **Deployment → ReplicaSet**. Inside: **initContainers** ran first; the **app container** is serving because
its **readiness** probe passed (**liveness** restarts it if it hangs, **startup** covers slow boot); a
**sidecar** (service-mesh **Envoy** or a log shipper) may share the pod. It has a **QoS class** from its
**requests/limits** (can be **OOMKilled** if it exceeds memory). Config comes from **ConfigMap**/**Secret**
(env or mounted volume) and the **downward API**; its **ServiceAccount** grants AWS access via **IRSA**
(temporary STS creds).

**(10) The app's downstream calls.** Each has its **own** network + auth path:
- **RDS/Aurora** (SQL) in a private subnet — the DB **SG allows the app SG** on `5432`; app uses a
  **connection pool**.
- **ElastiCache/Redis** — SG-gated; read-through/write-through caching.
- **S3** — via an **S3 Gateway Endpoint** (stays on AWS network) using the **IRSA** role.
- **SQS/Kafka** — offload slow work asynchronously to **worker** pods.
- **Secrets Manager / Vault** — fetch secrets (needs **kms:Decrypt**).
- **Other services** — internal ALB / Service DNS / **service mesh mTLS**.

**(11) Response path.** Back out: **Pod → Service → Ingress controller → ALB → (CloudFront) → client**.
Stateful **SG** return traffic is auto-allowed; a **NACL** needs an explicit ephemeral-port (`1024–65535`)
return rule.

---

## The surrounding context (senior interviewers pull on these threads)

### How did that pod get there? — CI/CD + GitOps
```text
Dev pushes → GitHub → GitHub Actions (build, unit tests, security scan)
   → build Docker image (multi-stage) → push to ECR (auth via OIDC, no static keys)
   → update image tag in the manifests/Helm repo (Git)
   → Argo CD detects the Git change → reconciles → EKS rolling update
   → readiness gates traffic; rollback = revert the Git commit
```

### How does the infrastructure exist? — Terraform / IaC
```text
Terraform (remote state in S3 + DynamoDB lock) provisions:
   VPC, subnets, route tables, IGW/NAT, Security Groups, ALB,
   EKS cluster + node groups (or Karpenter), IAM roles + IRSA/OIDC,
   RDS, ElastiCache, S3, ECR, Route 53 records, ACM certs
```

### How do you see what's happening? — Observability (3 pillars)
```text
Metrics: Prometheus scrapes pods/nodes → Grafana dashboards → Alertmanager
Logs:    app → stdout → Fluent Bit/Fluentd → CloudWatch Logs / Elasticsearch(Loki) → Kibana/Grafana
Traces:  OpenTelemetry / Jaeger — follow one request across services (trace ID)
Health:  liveness/readiness probes · ALB target health · CloudWatch alarms
```

### How does it stay up / scale?
- **HPA** scales **pods** on CPU/memory/custom metrics; **Karpenter/Cluster Autoscaler** scales **nodes**.
- **Multi-AZ** everything; **anti-affinity/topology spread** distributes replicas.
- **PodDisruptionBudget** protects availability during drains/upgrades.
- **Deployment strategies**: rolling (default), blue-green, canary.

---

## Component checklist (tick that you can explain each in one line)
DNS/Route 53 · CloudFront · WAF · Shield · ACM/TLS/SNI · ALB (listener/rule/target group/health check) ·
NLB · Security Group · NACL · route table · IGW · NAT · public/private subnet · Ingress · Ingress
controller · Gateway API · Service (ClusterIP/NodePort/LoadBalancer/Headless) · CoreDNS · kube-proxy
(iptables/IPVS) · EndpointSlice · kubelet · containerd · CNI (VPC CNI) · NetworkPolicy · scheduler ·
Deployment · ReplicaSet · StatefulSet · DaemonSet · Job/CronJob · Pod · initContainer · sidecar · probes
(liveness/readiness/startup) · requests/limits/QoS/OOMKilled · ConfigMap · Secret · downward API ·
ServiceAccount · IRSA/OIDC · RBAC · namespace · resource quota · HPA · Karpenter/Cluster Autoscaler · PDB ·
taints/tolerations · affinity/anti-affinity · topology spread · Helm/Kustomize · Argo CD/GitOps · ECR ·
RDS/Aurora · ElastiCache · S3 (+gateway endpoint) · SQS/Kafka · Secrets Manager/Vault · KMS · service mesh
(Envoy/mTLS) · Prometheus/Grafana/Alertmanager · Fluent Bit/CloudWatch/ELK · OpenTelemetry/Jaeger ·
Terraform (remote state/locking).

---

## The 60-second verbal version (memorize this shape)
> "A request resolves through **Route 53**, hits **CloudFront/WAF**, terminates **TLS** at the **ALB**
> (listener → rule → target group → healthy target). Inside the **VPC**, **security groups** gate it to the
> **Ingress controller**, which routes at L7 to a **Service**. **CoreDNS** gave the client the ClusterIP;
> **kube-proxy** DNATs it to a **ready pod** (via **EndpointSlices**) on a **node**, where the **kubelet**,
> **containerd**, and **VPC CNI** run it. The **pod** — placed by the **scheduler**, kept alive by a
> **Deployment**, configured by **ConfigMaps/Secrets**, identified via **IRSA** — calls **RDS**,
> **ElastiCache**, **S3** (gateway endpoint), and **SQS**, each gated by SGs or IAM. The response returns
> back up the chain. That pod got there via **GitHub Actions → ECR → Argo CD**; the infra via **Terraform**;
> and I'd observe all of it with **Prometheus/Grafana, logs, and traces**, scaling with **HPA + Karpenter**."
