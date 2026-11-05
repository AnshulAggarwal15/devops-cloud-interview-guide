<!-- #genai: AWS interview note - VPC design, subnets, peering, transit gateway, VPC endpoints, CIDR planning for the cloud round -->

## Question
How do you design a VPC and its subnets, and how do VPCs connect to each other and to AWS services privately?

### Short Explanation
This builds on the public/private subnet basics into **design**: CIDR planning, subnet tiers across AZs, and the connectivity options — **peering, Transit Gateway, VPC endpoints** — that interviewers use to test whether you've run real networks, not just labs.

## Answer
- A **VPC** is an isolated network with a CIDR (e.g. `10.0.0.0/16`), sliced into **subnets per AZ**; **route tables** decide public vs private.
- Standard layout: **public** subnets (ALB, NAT, bastion) + **private** subnets (app/EKS) + **isolated/DB** subnets (RDS, no internet) — each tier in **≥2 AZs**.
- Connect VPCs with **peering** (1:1, non-transitive) or **Transit Gateway** (hub for many VPCs/on-prem); reach AWS services privately with **VPC endpoints**.

## Detailed Explanation

### CIDR & subnet planning
- Pick a **VPC CIDR** big enough but non-overlapping with other VPCs/on-prem (peering/VPN break on overlap).
- Split into subnets by **tier × AZ**: e.g. public-a/public-b, app-a/app-b, db-a/db-b.
- Leave headroom — **you can't shrink a subnet**; add secondary CIDRs later if needed.

### Public vs private (recap)
- **Public** = route `0.0.0.0/0 → IGW` + public IP.
- **Private** = `0.0.0.0/0 → NAT` (outbound only) or no internet route (isolated).

### Connecting VPCs
| Option | Use it for | Note |
|---|---|---|
| **VPC Peering** | Connect **two** VPCs | **Non-transitive**; must not overlap CIDRs; mesh grows fast |
| **Transit Gateway (TGW)** | **Hub-and-spoke** for many VPCs + on-prem | Transitive routing, scales to hundreds; regional |
| **VPN / Direct Connect** | On-prem connectivity | VPN over internet; DX = dedicated line |
| **PrivateLink** | Expose/consume a **single service** privately | Interface endpoint to a specific service, not the whole VPC |

### VPC endpoints (reach AWS privately)
- **Gateway endpoint**: S3 and DynamoDB — free, added as a route-table entry, keeps traffic off the internet/NAT.
- **Interface endpoint (PrivateLink)**: an ENI in your subnet for most other services (ECR, Secrets Manager, SSM, etc.) — priced per hour + data.
- Big wins: **no NAT data charges** for that traffic and a tighter security posture.

### Things they probe
- **Peering is not transitive**: A–B and B–C does **not** give A–C → use TGW for a mesh.
- **CIDR overlap** is the classic peering/VPN failure.
- Put **RDS/EKS nodes in private subnets**, only ALB/NAT/bastion in public.
- **Flow Logs** for troubleshooting; **SGs (stateful)** and **NACLs (stateless)** control traffic — see the SG vs NACL note.

## Summary
- Design = **CIDR plan + subnet tiers (public/private/isolated) across ≥2 AZs**, route tables decide reachability.
- **Peering** = simple 1:1 (non-transitive); **Transit Gateway** = scalable hub for many VPCs/on-prem.
- **Gateway endpoints** (S3/DynamoDB, free) and **interface endpoints/PrivateLink** reach AWS services without the internet or NAT.

## Related
- [VPC, subnets & routing](./101-vpc-subnets-routing.md)
- [Security Groups vs NACLs](./102-security-groups-vs-nacls.md)
- [NAT and when it's used](./02-aws-nat-and-when-used.md)
