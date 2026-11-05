<!-- #genai: AWS interview note - EKS migration strategy, self-managed k8s to EKS, cluster upgrades, blue-green, data/stateful migration for the cloud round -->

## Question
How would you migrate to Amazon EKS — from self-managed Kubernetes (or another cluster), and how do you cut over with minimal downtime?

### Short Explanation
"Migration" usually means one of: **self-managed k8s → EKS**, **on-prem/other cloud → EKS**, or **EKS cluster → new EKS cluster** (major version jump). Interviewers want a **phased, low-risk plan** — assess, stand up in parallel, migrate stateless first, handle stateful data carefully, shift traffic gradually, and keep rollback ready.

## Answer
- Prefer a **blue/green cluster** approach: build the new EKS cluster **alongside** the old one, replicate workloads, then **shift traffic via DNS/load balancer** — safer than an in-place mutation.
- Migrate in order: **stateless apps first**, then **stateful/data**, keeping both clusters serving until the new one is proven.
- Automate with **IaC (Terraform/eksctl) + GitOps (Argo CD/Flux)** so the target is reproducible and rollback is a traffic switch.

## Detailed Explanation

### 1. Assess & plan
- Inventory workloads, **CRDs, operators, ingress, storage classes, cluster add-ons** (CNI, DNS, ingress controller), and cloud dependencies.
- Map identities: replace static creds with **IRSA / EKS Pod Identity**.
- Check **Kubernetes version skew** and deprecated APIs (`kubectl` deprecations, Pluto/kubent) before moving manifests.

### 2. Stand up the target EKS cluster
- Provision with **Terraform/eksctl**: managed node groups or **Fargate/Karpenter**, VPC CNI, CoreDNS, **AWS Load Balancer Controller**, EBS/EFS CSI drivers.
- Recreate namespaces, RBAC, network policies, secrets (via **External Secrets Operator**), and add-ons.
- Wire **GitOps** so apps deploy declaratively to the new cluster.

### 3. Migrate workloads (stateless first)
- Deploy stateless Deployments/Services to EKS; validate with synthetic + smoke tests.
- Rebuild images for the right arch (watch **arm64/Graviton** vs amd64) and push to **ECR**.
- Recreate **Ingress** on the ALB controller; get certs via ACM.

### 4. Handle stateful & data
- Databases: prefer **managed RDS/Aurora** outside the cluster; migrate with **DMS**, snapshots, or replication + cutover.
- In-cluster PVs: re-provision **EBS/EFS** via CSI; move data with volume snapshots, `restic`/Velero backups, or app-level replication.
- Use **Velero** to back up/restore cluster objects and PVs between clusters.

### 5. Cut over with minimal downtime
- Run both clusters in parallel; shift traffic gradually with **Route 53 weighted records** (or a global ALB) — canary a small % first.
- Watch SLOs/dashboards; if healthy, ramp to 100%; if not, **shift weight back** (instant rollback).
- Decommission the old cluster only after a soak period.

### EKS version upgrades (a related "migration")
- In-place: upgrade **control plane** one minor version at a time, then **node groups**, then add-ons — respecting version skew.
- Or **blue/green node groups**: add new nodes, cordon/drain old ones with **PodDisruptionBudgets** so replicas stay available.

### Things they probe
- **Never big-bang.** Parallel run + gradual traffic shift + tested rollback.
- **Stateful data** is the hard part — plan replication/backup and a cutover window.
- Identity (**IRSA**), storage (**CSI**), and ingress (**ALB controller**) are the usual migration gotchas.

## Summary
- Migrate via **blue/green clusters**: build EKS in parallel with IaC + GitOps, move **stateless first**, then **stateful/data** (DMS/Velero/snapshots).
- Cut over with **Route 53 weighted routing / canary**, monitor SLOs, keep rollback = shift weight back.
- Fix the common gotchas up front: **API deprecations, IRSA identity, CSI storage, ALB ingress, image architecture.**

## Related
- [ECS vs EKS](./105-ecs-vs-eks.md)
- [IRSA](./104-irsa.md)
- [Deployment strategies](../08-kubernetes/16-all-deployment-strategies.md)
- [ASG & scaling](./115-asg-and-scaling.md)
