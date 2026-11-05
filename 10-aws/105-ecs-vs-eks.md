<!-- #genai: AWS interview note - compute options (EC2 vs Lambda vs ECS vs EKS) and how to choose, for the Docusign R2 cloud round -->

## Question
Compare AWS compute options — **EC2, Lambda, ECS, EKS** — and explain how you'd choose between them.

### Short Explanation
Think of a **spectrum from "you manage everything" to "AWS manages everything"**:
**EC2** (raw servers) → **ECS/EKS** (container orchestration) → **Lambda** (serverless functions).
More control on the left; less operational overhead on the right.

---

## The spectrum (the framing that wins the question)
```text
MORE control / more ops  ───────────────────────────────►  LESS ops / more managed
   EC2            EKS              ECS            Fargate           Lambda
(raw VMs)   (managed K8s)   (AWS orchestrator)  (serverless      (serverless
                                                  containers)      functions)
```

| | EC2 | ECS | EKS | Lambda |
|---|---|---|---|---|
| **What it is** | Virtual machines | AWS container orchestrator | Managed Kubernetes | Event-driven functions |
| **Unit** | Server/VM | Task (container) | Pod | Function invocation |
| **You manage** | OS, patching, scaling, runtime | Task defs (Fargate = no servers) | K8s + nodes (Fargate = no nodes) | Just your code |
| **Scaling** | Auto Scaling Groups | Service auto scaling | HPA + Karpenter/CA | Automatic, per-request |
| **Billing** | Per running instance (hour/sec) | Per task / Fargate resources | Per node + **hourly control-plane fee** | Per request + duration (**scales to zero**) |
| **Startup** | Minutes | Seconds | Seconds | Milliseconds (warm) |
| **Best for** | Full control, legacy, stateful, GPU | Simple container apps on AWS | Portable/rich K8s ecosystem | Short, spiky, event-driven work |
| **Portability** | AWS (VM is generic) | AWS-only | Portable (Kubernetes) | AWS-only (proprietary) |

---

## When to choose each
### EC2
- You need **full control** of the OS/runtime, special kernels, GPU, or licensing.
- **Long-running, steady** workloads; lifting-and-shifting legacy apps.
- Trade-off: **you** own patching, scaling, and availability.

### Lambda
- **Event-driven / spiky / short** tasks: API backends, file-processing on S3 upload, cron, glue between
  services, automation.
- **Scales to zero** — pay nothing when idle; scales out automatically.
- Trade-offs: **15-min max** runtime, **cold starts**, size limits, harder for long or stateful jobs. In a
  VPC it needs ENIs + NAT/endpoints for outbound.

### ECS
- You want to run **containers simply on AWS** with minimal orchestration overhead.
- **ECS + Fargate** = serverless containers (no servers to manage) — a very common low-ops choice.
- Trade-off: **AWS-only**, smaller ecosystem than Kubernetes.

### EKS
- You need the **Kubernetes ecosystem**: Helm, **Argo CD / GitOps**, operators, HPA/KEDA, service mesh, CRDs.
- You want **portability** (multi-cloud / hybrid / avoid lock-in) or already have K8s skills/manifests.
- Trade-offs: **more moving parts** and a **per-cluster control-plane fee**.

---

## Fargate — the cross-cutting option
**Fargate** is a **serverless compute engine** for **both ECS and EKS**: you run containers **without
managing nodes/servers**. Choose it when you want containers but not the ops of a node fleet; choose
self-managed EC2 nodes when you need node-level control, GPUs, or lower cost at scale.

## Good interview framing
> "I place them on a control-vs-ops spectrum. **EC2** when I need full control or it's a steady/legacy
> workload. **Lambda** for short, event-driven, spiky work that should scale to zero. For containers, **ECS
> (especially with Fargate)** when I want the simplest AWS-native path, and **EKS** when I need Kubernetes'
> portability and ecosystem — GitOps with Argo CD, Helm, operators — accepting more ops and the control-plane
> fee. For a platform team standardizing on GitOps, EKS usually wins."

## Summary
- **Spectrum:** EC2 (most control) → ECS/EKS (containers) → Lambda (least ops, scales to zero).
- **EC2** = raw VMs, full control, you patch/scale. **Lambda** = serverless functions, event-driven, 15-min
  cap, cold starts.
- **ECS** = simple AWS-native containers (free control plane); **EKS** = managed Kubernetes (portable, rich
  ecosystem, paid control plane, more ops).
- **Fargate** = serverless containers for either ECS or EKS (no node management).
