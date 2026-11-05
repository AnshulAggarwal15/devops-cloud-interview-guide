<!-- #genai: AWS interview note - provisioning a secure, production-grade EKS cluster from scratch with Terraform; blocks, parameters, private control plane, IAM, add-ons -->

## Provisioning a Production EKS Cluster From Scratch (Terraform)

### Question
"Set up an EKS cluster from scratch — what do you configure, in what order? What **Terraform blocks and parameters** do you use? Make it **production-grade and secure** (e.g. the control plane must not be reachable from the internet)."

### Short Explanation
Interviewers want a **layered build order** and the **security parameters** that make it production-grade — especially a **private API endpoint**, **private subnets for nodes**, **least-privilege IAM**, and **managed add-ons**. You provision it with **Terraform** (or `eksctl`), never by hand.

## Answer
Build it in layers: **networking → control plane → node capacity → IAM/access → add-ons → security → ingress/observability**.
- **Networking:** a VPC with a planned **CIDR**, **private + public subnets across ≥2 AZs**, route tables, **NAT** (egress), and security groups.
- **Control plane:** the EKS cluster with the **API endpoint set to private** (or private + tightly-scoped `public_access_cidrs`), **KMS envelope encryption** for secrets, and control-plane logging.
- **Nodes:** **managed node groups** (or **Karpenter**) in the **private** subnets, mixed On-Demand/Spot, autoscaling.
- **IAM/access:** **cluster role**, **node role**, **IRSA/Pod Identity** for workloads, and **EKS access entries** (or `aws-auth`) mapping human IAM identities to RBAC.
- **Add-ons:** **VPC CNI, CoreDNS, kube-proxy**, EBS/EFS CSI — as managed add-ons.
- **Security & edge:** default-deny **NetworkPolicies**, **Pod Security**, **AWS Load Balancer Controller**, TLS, observability.

## Detailed Explanation

### Build order & the Terraform blocks
```text
VPC ─► subnets (private/public) ─► IGW/NAT/route tables ─► SGs
  ─► EKS control plane (aws_eks_cluster) ─► node groups (aws_eks_node_group)
  ─► IAM roles + OIDC provider (IRSA) ─► access entries
  ─► add-ons (aws_eks_addon) ─► LB controller / observability
```
| Layer | Terraform blocks (typical) | Key parameters |
|---|---|---|
| **Networking** | `aws_vpc`, `aws_subnet`, `aws_route_table`, `aws_nat_gateway`, `aws_security_group` | VPC **`cidr_block`** (e.g. `10.0.0.0/16`), per-AZ subnet CIDRs, subnet **tags** for LB/CNI |
| **Control plane** | `aws_eks_cluster` | `vpc_config` with **`endpoint_private_access = true`**, **`endpoint_public_access = false`** (or scoped `public_access_cidrs`), **`encryption_config`** (KMS), `enabled_cluster_log_types` |
| **Nodes** | `aws_eks_node_group` (or Karpenter) | instance types, **`scaling_config`** (min/desired/max), **private `subnet_ids`**, capacity type (SPOT/ON_DEMAND) |
| **IAM** | `aws_iam_role`, `aws_iam_openid_connect_provider` | cluster role, node role, **OIDC provider** for IRSA |
| **Access** | `aws_eks_access_entry` (or `aws-auth`) | map IAM principals → K8s groups → RBAC |
| **Add-ons** | `aws_eks_addon` | `vpc-cni`, `coredns`, `kube-proxy`, `aws-ebs-csi-driver` |

### The security parameters that make it "production-grade"
- **Private control-plane endpoint** — set **`endpoint_public_access = false`** (or restrict `public_access_cidrs` to your VPN/office). This is the "control plane must not be reachable from the internet" ask. `kubectl` then reaches the API **from inside the VPC** (bastion/VPN/PrivateLink).
- **Nodes in private subnets** — no public IPs; egress via **NAT**; inbound only from the ALB/control plane via **security groups**.
- **KMS secrets encryption** — `encryption_config` envelope-encrypts Kubernetes Secrets in etcd.
- **Least-privilege IAM** — narrow cluster/node roles; **IRSA** so pods get their own scoped roles (not the node role).
- **Control-plane logging** — enable `api`, `audit`, `authenticator` logs to CloudWatch.
- **Security groups** — cluster SG + node SG; allow only required flows; keep the API SG tight.
- **Network segmentation** — default-deny **NetworkPolicies**, namespaces per env with **ResourceQuotas/LimitRanges**.

### CIDR planning (they often drill this)
- Pick a **non-overlapping** VPC CIDR (overlap breaks peering/VPN). `/16` = 65,536 IPs; carve **`/24`** subnets per tier per AZ.
- Leave headroom — the **VPC CNI assigns real VPC IPs to pods**, so subnets drain fast; size subnets generously or use secondary CIDRs / prefix delegation.
- (CIDR/subnetting basics: see the networking primer.)

### "Increase the size" — of *what*? (the classic follow-up)
Don't say "cluster size." Identify the constrained layer:
| "Size" | What changes |
|---|---|
| **Node/EC2 size** | Bigger instance type (`m6i.large → m6i.xlarge`) — more CPU/RAM per node |
| **Node group size** | More nodes (desired 3 → 5) — via ASG/Karpenter |
| **Pod resources** | `requests`/`limits` (500m/512Mi → 1/1Gi) |
| **EBS volume** | PVC storage (50Gi → 100Gi) |
| **Control plane** | You **don't** resize it — AWS manages/scales it |
*"I'd check whether the bottleneck is pod requests, node capacity, node count, or storage, and scale that layer — not some single 'cluster size'."*

### What's *inside* the cluster (if asked conceptually)
- **Control plane (AWS-managed on EKS):** **API server, etcd, scheduler, controller manager.**
- **Worker nodes:** **kubelet, kube-proxy, container runtime (containerd).**
- **kube-proxy, CoreDNS, VPC CNI are managed EKS add-ons** — you don't install them per node by hand. (See cluster architecture note.)

### Things they probe
- Provision with **Terraform/IaC**, not the console; store state in **S3 + locking**.
- **Private endpoint + private-subnet nodes + KMS + IRSA + least privilege** = the production security story.
- Know **which layer to scale** and that the **control plane is managed**.

## Summary
- Build **networking → control plane → nodes → IAM/access → add-ons → security/edge**, all in **Terraform**.
- Production security = **private API endpoint**, **nodes in private subnets (NAT egress)**, **KMS secrets encryption**, **least-privilege IAM + IRSA**, **control-plane logging**, tight **security groups**, default-deny **NetworkPolicies**.
- Plan a **non-overlapping CIDR** (CNI burns real VPC IPs), and when asked to "increase size," scale the **right layer** (pod/node/count/storage) — the control plane is AWS-managed.

## Related
- [VPC & subnet design](./116-vpc-and-subnet-design.md) · [CIDR & subnetting basics](../04-networking/10-cidr-and-subnetting-basics.md)
- [IRSA](./104-irsa.md) · [Cluster hardening](../08-kubernetes/40-cluster-hardening.md)
- [EKS migration](./120-eks-migration.md) · [Cluster architecture](../08-kubernetes/01-cluster-architecture.md)
