# K8s / Terraform / Prometheus / Helm / AWS / Docker — Coverage Map

A scenario-focused interview set (Kubernetes-heavy, plus Terraform, Prometheus/Grafana, Helm, S3/IAM, Docker). Each question maps to its answer note. Drill it: cover the right column, answer out loud, open the note only when you stumble.

**Legend:** ✅ dedicated note · ⭐ added while filling this bank · ⚠️ answered inline below

## 🔹 Kubernetes
| Question | Where |
|---|---|
| What is kube-proxy? | ✅ [Service relates to kube-proxy](../08-kubernetes/08-service-relates-to-kubeproxy.md) |
| Deployment vs StatefulSet | ✅ [Deployment/StatefulSet/DaemonSet](../08-kubernetes/42-deployment-statefulset-daemonset.md) |
| Types of Kubernetes Services | ✅ [Types of services](../08-kubernetes/05-types-of-services.md) |
| Headless Service & how it works | ✅ [Headless service](../08-kubernetes/10-headless-service.md) |
| NodePort Service | ✅ [NodePort vs LB](../08-kubernetes/07-nodeport-vs-lb.md) |
| Request flow: domain → Pod via DNS + Services | ⚠️ inline below (+ [CoreDNS deep dive](../08-kubernetes/38-coredns-deep-dive-and-troubleshooting.md) · [request lifecycle](../docusign-request-lifecycle-handbook.md)) |
| Move workloads from unhealthy → healthy node | ⭐ [Cordon/drain vs taint](../08-kubernetes/47-cordon-drain-vs-taint.md) |
| Taint vs Cordon | ⭐ [Cordon/drain vs taint](../08-kubernetes/47-cordon-drain-vs-taint.md) |
| How to restart a Deployment | ⚠️ inline below |
| Application health check | ✅ [Liveness vs readiness (+ startup)](../08-kubernetes/20-liveness-vs-readiness.md) |
| Types of probes | ✅ [20 – probes](../08-kubernetes/20-liveness-vs-readiness.md) |
| Which probe acts first (startup/readiness/liveness) | ✅ [20 – startup runs first, disables the others](../08-kubernetes/20-liveness-vs-readiness.md) |
| Common CrashLoopBackOff reasons | ✅ [CrashLoopBackOff](../08-kubernetes/19-crashloopbackoff.md) |

## 🔹 Terraform
| Question | Where |
|---|---|
| What is a Terraform state file? | ✅ [Role of the state file](../06-terraform/03-role-of-statefile.md) |
| How does state locking work? | ✅ [Concurrent state update / locking](../06-terraform/06-concurrent-statefile-update.md) |
| What is Terraform drift? | ⭐ [Terraform drift](../06-terraform/17-terraform-drift.md) |
| `terraform import` and when to use it | ✅ [Import blocks & state recovery](../06-terraform/103-import-blocks-and-state-recovery.md) |
| Why S3 as a Terraform backend? | ✅ [State file management](../06-terraform/05-statefile-management.md) (+ inline below) |
| Advantages of a remote backend | ✅ [State file management](../06-terraform/05-statefile-management.md) |

## 🔹 Prometheus & Grafana
| Question | Where |
|---|---|
| Hands-on with Prometheus & Grafana | ✅ [Observability tools](../09-observability/07-observability-tools.md) |
| Collect application-level metrics | ✅ [Metrics to scrape](../09-observability/03-metrics-to-scrape-with-prometheus.md) · [Emit custom metrics](../09-observability/02-emit-custom-logs-and-metrics.md) |
| Collect application-level logs | ✅ [Logs, metrics, traces](../09-observability/05-logs-metrics-traces.md) · [Emit custom logs](../09-observability/02-emit-custom-logs-and-metrics.md) |
| Data source used with Grafana | ✅ [Multi-cluster (Thanos/Mimir/Prometheus as source)](../09-observability/12-multi-cluster-central-observability.md) |
| How does Prometheus collect metrics? | ✅ [Push vs pull (Prometheus scrapes /metrics)](../09-observability/06-push-vs-pull-monitoring.md) |

## 🔹 Helm
| Question | Where |
|---|---|
| The `range` function in Helm | ⭐ [Helm essentials](../08-kubernetes/46-helm-essentials.md) |
| Values vs parameters in Helm | ⭐ [Helm essentials](../08-kubernetes/46-helm-essentials.md) |
| How to pass values during deployment | ⭐ [Helm essentials (`-f` / `--set`)](../08-kubernetes/46-helm-essentials.md) |

## 🔹 AWS / S3 / IAM
| Question | Where |
|---|---|
| Give access to a specific S3 file to another org/account | ⭐ [S3 access: policy vs IAM vs pre-signed](../10-aws/32-s3-access-bucket-policy-iam-presigned.md) |
| Bucket policy vs IAM policy vs pre-signed URL | ⭐ [32](../10-aws/32-s3-access-bucket-policy-iam-presigned.md) |
| Temporary access to a specific S3 object | ⭐ [32 – pre-signed URL](../10-aws/32-s3-access-bucket-policy-iam-presigned.md) |

## 🔹 Docker
| Question | Where |
|---|---|
| List running containers | ⚠️ `docker ps` — [Docker commands](../07-docker/11-docker-commands.md) |
| List all containers | ⚠️ `docker ps -a` — [Docker commands](../07-docker/11-docker-commands.md) |
| Common reasons containers keep restarting | ✅ [Container exits immediately](../07-docker/01-docker-container-exits-immediately.md) (+ K8s [CrashLoop](../08-kubernetes/19-crashloopbackoff.md)) |

---

## ⚠️ Inline answers for the partials

**Request flow: domain name → Pod (DNS + Kubernetes Services)**
```text
Client → public DNS (Route 53) resolves the domain → ALB / Ingress (edge, L7, TLS)
  → Ingress routes by host/path to a Service (ClusterIP)
  → kube-proxy (iptables/IPVS) DNATs the ClusterIP to a healthy Pod IP (from the EndpointSlice)
  → Pod serves the request
Inside the cluster, service-to-service uses CoreDNS: svc.ns.svc.cluster.local → ClusterIP → kube-proxy → Pod.
```
So: **external DNS gets you to the edge (ALB/Ingress); the Service + kube-proxy + EndpointSlice get you to a Pod; CoreDNS handles in-cluster name resolution.** Full internal detail in [CoreDNS deep dive](../08-kubernetes/38-coredns-deep-dive-and-troubleshooting.md); full external path in the [request lifecycle handbook](../docusign-request-lifecycle-handbook.md).

**How to restart a Deployment (no YAML change)**
```bash
kubectl rollout restart deployment/<name>      # rolling restart, zero-downtime
kubectl rollout status deployment/<name>       # watch it
kubectl rollout undo   deployment/<name>       # roll back if needed
```
`rollout restart` triggers a **new rollout** (pods recreated per the RollingUpdate strategy) without editing the spec — the clean way to pick up a rotated Secret/ConfigMap or clear bad state. Avoid `kubectl delete pod` for this.

**Why S3 as a Terraform backend / remote-backend advantages**
- **Shared state** (team sees one source of truth, not local files), **locking** via DynamoDB (no concurrent-apply corruption), **versioning** (recover a previous state), **encryption at rest** (KMS), **durability** (11 nines), and **access control** (IAM). It's cheap, native to AWS, and integrates with CI. (See [state management](../06-terraform/05-statefile-management.md).)

**Docker ps quick-hits**
- `docker ps` → **running** containers; `docker ps -a` → **all** (incl. exited). Restart reasons: app exits/crashes, bad `CMD`/`ENTRYPOINT`, missing config/env, OOM, failing healthcheck, dependency not ready. (See [container exits immediately](../07-docker/01-docker-container-exits-immediately.md).)

---

## How to use this bank
1. Cover the right column; answer each in 60–90 seconds out loud.
2. Stumble → open the note, re-learn the mechanism, retry tomorrow.
3. Prioritize the ⭐ (were gaps) and anything Kubernetes.

> Other banks & transcripts: [Common AWS DevOps/SRE](./05-common-aws-devops-sre-questions.md) · [20 real questions](./06-twenty-real-interview-questions.md) · [Nielsen](./01-nielsen.md) · [Amagi](./02-amagi.md) · [Jio](./03-jio-cloud-security.md) · [MoveInSync](./04-moveinsync.md)
