## Advanced Autoscaling — Custom Metrics, KEDA, Multiple Triggers & Stabilization

### Question  
You have a stateless app and want to autoscale it — but not just on CPU/memory. How do you scale on **custom application metrics** (from Prometheus) or **events** (queues)? What happens when **multiple autoscalers / triggers** want to scale at the same time, and how do you stop **over-provisioning** when pods are slow to start?

### Short explanation of the question  
This goes past "use an HPA." Interviewers push on: (1) scaling on **custom/external metrics**, (2) **event-driven** scaling with **KEDA**, (3) **conflicts** when two autoscalers target one Deployment or one KEDA ScaledObject has many triggers, and (4) **thrashing/over-provisioning** when startup lag makes the metric pile up. Know all four.

---

### Answer  
- Deploy the app as a **stateless Deployment** behind a Service, with **requests/limits** and a **readiness probe**, then attach an **HPA**.
- To scale on **custom app metrics**, expose them to the HPA via the **custom/external metrics API** — typically the **Prometheus Adapter** (registers `custom.metrics.k8s.io` / `external.metrics.k8s.io`). The HPA references that metric instead of CPU.
- The other approach is **KEDA** — event-driven autoscaling from **external sources** (Prometheus, RabbitMQ, Kafka, SQS, cron, etc.). KEDA can also scale to **zero**.
- **Conflict rule:** never point **two autoscalers at the same Deployment** — they fight and thrash. Within **one** KEDA `ScaledObject` with multiple triggers, KEDA computes each trigger's desired replicas and takes the **maximum** (scaling out is safe; the highest demand wins).
- **Over-provisioning from startup lag** is solved with the HPA **stabilization window / scaling behavior** (and KEDA `cooldownPeriod`/`pollingInterval`) plus a **readiness probe**, not by lowering `maxReplicas`.

---

### Detailed explanation of the answer for readers’ understanding

### 🚀 1. Deploy stateless + basic autoscale
- **Deployment** (N replicas) + **Service**; state lives in RDS/S3/cache, not the pod.
- Set **resource `requests`** (HPA percentages are relative to requests) and a **readiness probe** (so traffic/metrics only count ready pods).
- Pair **HPA** (more pods) with the **Cluster Autoscaler / Karpenter** (more nodes) so pods aren't stuck `Pending`.
- (Basics: see [horizontal vs vertical scaling](./34-horizontal-vs-vertical-scaling.md).)

### 📈 2. Scale on custom Prometheus metrics (HPA + Prometheus Adapter)
CPU/memory HPA can't see "requests per second" or "queue depth." Bridge Prometheus → HPA with the **Prometheus Adapter**, which serves the **custom metrics API**:
```text
Prometheus ──> Prometheus Adapter ──> custom.metrics.k8s.io ──> HPA ──> Deployment
```
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web
  minReplicas: 2
  maxReplicas: 50
  metrics:
  - type: Pods                      # a custom per-pod metric
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "100"
```
**How does the HPA "find" the custom metric?** It doesn't read Prometheus directly. The Prometheus Adapter **registers the metric name** with the Kubernetes **aggregated metrics API** (`custom.metrics.k8s.io` for object/pod metrics, `external.metrics.k8s.io` for external ones). The HPA controller queries that API by name; the adapter translates the request into a PromQL query. So the metric must be **exposed by the adapter's config rules** — that's what makes it discoverable.

### ⚡ 3. Event-driven scaling with KEDA (the "other approach")
**KEDA** watches external sources and drives scaling via a **`ScaledObject`**. Important: **KEDA creates and manages an HPA under the hood** — you don't write the HPA yourself.
```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: worker
spec:
  scaleTargetRef:
    name: worker                    # the Deployment
  minReplicaCount: 0                # KEDA can scale to ZERO
  maxReplicaCount: 100
  cooldownPeriod: 300               # wait before scaling back down
  pollingInterval: 15
  triggers:
  - type: prometheus
    metadata: { query: "sum(rate(http_requests_total[1m]))", threshold: "100" }
  - type: rabbitmq
    metadata: { queueName: "jobs", queueLength: "50" }
```

### 🥊 4. Conflicts — the part they probe hard
- **Two autoscalers on ONE Deployment** (e.g. a hand-written HPA **and** a KEDA ScaledObject, which itself makes an HPA): **don't do this.** Two controllers set `.spec.replicas` on the same target and **fight** — the Deployment flaps up and down (thrashing). Pick one owner per Deployment.
- **One KEDA ScaledObject with MANY triggers** (Prometheus + RabbitMQ + Spark): this is fine and expected. KEDA evaluates **each trigger's desired replica count** and feeds them into a single HPA, which takes the **maximum**.
  - Example: trigger A wants **15**, trigger B wants **100**, trigger C wants **5** → the result is **100** (the max). Scaling **out** always wins because under-provisioning is riskier than over-provisioning; a "scale to 5" trigger can't shrink you while another demands 100. No cluster error — it's by design.

### 🛑 5. Stop over-provisioning when pods start slowly
Scenario: metric says "scale to 50," but pods take ~60s to become ready. If the HPA keeps reacting every cycle while the new pods aren't `Ready` yet, the metric stays high and it tries to **pile on more pods** → overshoot.

Fixes (in order):
- **Readiness probe** — unready pods don't receive traffic and shouldn't count toward the load signal, so the metric reflects reality once they're actually serving.
- **HPA `behavior` / stabilization window** — cap how fast and how much it scales, and smooth flapping:
```yaml
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60      # consider recent recommendations, don't overreact
      policies:
      - type: Pods
        value: 10
        periodSeconds: 60                 # add at most 10 pods/min
    scaleDown:
      stabilizationWindowSeconds: 300      # scale down slowly to avoid flapping
```
- **KEDA** `cooldownPeriod` + `pollingInterval` play the same role for event-driven scaling.
- **Not** the fix: lowering `maxReplicas`. If 50 is the legitimate target and `maxReplicas` is 500, capping at 50 is the *stabilization window* doing its job — the ceiling isn't what's holding it; the rate/stabilization controls are. `maxReplicas` is a safety ceiling, not a smoothing tool.

### ⚠️ Things they probe
- HPA reads metrics from the **metrics API**, not Prometheus directly — the **adapter** (or KEDA's metrics adapter) is the bridge.
- **One autoscaler per workload.** KEDA already owns an HPA; don't add a second.
- **Max wins** across KEDA triggers; scaling out beats scaling in.
- Startup lag → **readiness probe + stabilization window**, not a smaller max.

---

### 🧪 Real-World Example

> “Our workers scaled on **RabbitMQ queue depth** and **request rate** via a single **KEDA ScaledObject** (min 0, max 100). Both triggers fed one HPA, which took the **max** desired replicas. When a deploy caused slow starts, pods scaled way past need — we added a **readiness probe** and a **scaleUp stabilization window** (max 10 pods/min), and the overshoot disappeared. We were careful to have **only KEDA** manage that Deployment — an extra manual HPA had earlier caused thrashing.”

---

### Key takeaway

> “Custom-metric scaling = **HPA + Prometheus Adapter** exposing `custom.metrics.k8s.io`; event-driven = **KEDA ScaledObject** (which manages its own HPA and can scale to zero). **One autoscaler per Deployment** — two will fight. Across **KEDA triggers, the maximum desired replicas wins**. Stop startup-lag overshoot with a **readiness probe + HPA stabilization window / KEDA cooldown**, not by shrinking `maxReplicas`.”

### Related
- [Horizontal vs vertical scaling (HPA/VPA/Cluster Autoscaler basics)](./34-horizontal-vs-vertical-scaling.md)
- [Metrics to scrape with Prometheus](../09-observability/03-metrics-to-scrape-with-prometheus.md)
- [Liveness vs readiness probes](./20-liveness-vs-readiness.md)
- [Requests and limits](./31-requests-and-limits.md)
