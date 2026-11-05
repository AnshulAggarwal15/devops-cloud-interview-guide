<!-- #genai: AWS interview note - Auto Scaling Groups, launch templates, scaling policies, horizontal vs vertical scaling for the cloud round -->

## Question
Explain Auto Scaling Groups (ASG) and horizontal vs vertical scaling on AWS.

### Short Explanation
An **ASG** keeps a fleet of EC2 instances at a **desired capacity**, replaces unhealthy ones, and scales in/out on demand. Behind it sits the core idea interviewers want: **horizontal (scale out — more instances)** vs **vertical (scale up — bigger instance)**, and why you usually prefer horizontal for stateless tiers.

## Answer
- An **ASG** manages EC2 instances between a **min / desired / max** using a **launch template**, spreads them across AZs, and replaces failed instances via **health checks**.
- **Horizontal scaling (scale out)** = add more instances → resilient, near-unlimited, no downtime. This is what an ASG does.
- **Vertical scaling (scale up)** = move to a larger instance type → simple but bounded by max instance size and usually needs a **stop/replace**.

## Detailed Explanation

### What an ASG gives you
- **Desired/min/max capacity** with automatic **replacement** of unhealthy instances (EC2 or ELB health checks).
- **Multi-AZ** distribution for HA; integrates with **ALB/NLB target groups**.
- **Launch template** defines AMI, instance type, user data, security groups, IAM instance profile.
- **Lifecycle hooks** for graceful boot/drain; **termination policies** decide who goes on scale-in.

### Scaling policies
| Policy | Behavior |
|---|---|
| **Target tracking** | Hold a metric at target (e.g. CPU 60%, ALB requests/target) — the go-to |
| **Step scaling** | Add/remove N instances per alarm threshold band |
| **Simple scaling** | One adjustment per alarm (older) |
| **Scheduled** | Scale on a known time pattern (business hours, batch windows) |
| **Predictive** | ML-based forecast pre-scales ahead of demand |

### Horizontal vs vertical
| | Horizontal (out) | Vertical (up) |
|---|---|---|
| What changes | **Number** of instances | **Size** of the instance |
| Downtime | None (add/remove) | Usually **stop → resize → start** |
| Ceiling | Very high | Bounded by largest instance type |
| Resilience | Higher (spread of instances) | Single bigger box = bigger blast radius |
| Best for | **Stateless** web/API tiers | Hard-to-distribute or single-instance workloads (some DBs) |

**Instinct they want:** scale the **stateless** tier **horizontally** with an ASG behind a load balancer; reserve **vertical** for stateful things that can't be replicated, and protect databases with replicas/caching instead of just a bigger box.

### Things they probe
- ASG uses **health checks** to self-heal — turn on **ELB health checks**, not just EC2, so it replaces instances failing at the app layer.
- Use **warm pools / lifecycle hooks** to cut cold-start time.
- Combine ASG with **Spot + On-Demand** mixed instances for cost.
- On EKS/K8s the analog is **HPA (out)**, **VPA (up)**, and **Cluster Autoscaler / Karpenter** (nodes) — see the Kubernetes note.

## Summary
- **ASG** = self-healing, multi-AZ EC2 fleet between min/desired/max, driven by **scaling policies** (target tracking is default).
- **Horizontal = more instances (preferred for stateless, no downtime, resilient); Vertical = bigger instance (bounded, needs replace).**
- Scale compute horizontally; protect the stateful DB tier separately.

## Related
- [High availability & autoscaling](./107-high-availability-and-autoscaling.md)
- [ASG not launching EC2 — troubleshooting](./12-asg-not-launching-ec2.md)
- [Horizontal vs vertical scaling in Kubernetes](../08-kubernetes/34-horizontal-vs-vertical-scaling.md)
