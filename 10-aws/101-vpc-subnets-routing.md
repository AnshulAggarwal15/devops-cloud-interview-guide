<!-- #genai: AWS interview note - VPC, subnets, route tables, IGW, NAT for the Docusign R2 cloud round -->

## Question
Explain a VPC and how traffic gets in and out of public vs private subnets.

### Short Explanation
A **VPC** is your own isolated virtual network in AWS. You carve it into **subnets** (each in one Availability Zone). Whether a subnet is "public" or "private" is decided entirely by its **route table**, not by a checkbox.

## Answer

| Concept | Role |
|---|---|
| **VPC** | Isolated network; you define the CIDR block (e.g. `10.0.0.0/16`) |
| **Subnet** | A slice of the VPC CIDR in a single AZ (e.g. `10.0.1.0/24`) |
| **Route table** | Rules that decide where traffic for a destination goes |
| **Internet Gateway (IGW)** | VPC-level door to the internet; makes a subnet *public* |
| **NAT Gateway** | Lets *private* subnets make **outbound** internet calls, no inbound |

- **Public subnet** = route table has `0.0.0.0/0 -> igw-xxxx`.
- **Private subnet** = route table has `0.0.0.0/0 -> nat-xxxx` (or no internet route at all).

## Detailed Explanation

### Making a subnet public
A subnet is public when its route table sends `0.0.0.0/0` to the **Internet Gateway**, and the instance has a **public IP**.
```text
Destination     Target
10.0.0.0/16     local          # intra-VPC traffic, always present
0.0.0.0/0       igw-0abc123     # this line makes it public
```

### Private subnet outbound via NAT
Private subnets have **no inbound** path from the internet. For **outbound** (e.g. `yum`/`apt`, calling an external API), traffic routes to a **NAT Gateway** that lives in a *public* subnet:
```text
Destination     Target
10.0.0.0/16     local
0.0.0.0/0       nat-0def456     # outbound only; nobody can initiate inbound
```

### Key facts they probe
- **NAT Gateway is one-way**: outbound-initiated only. Inbound connections cannot start through it.
- **NAT Gateway is AZ-scoped** and costs money + charges per GB — for HA you run one per AZ.
- Typical layout: **public subnets** hold ALBs / NAT / bastion; **private subnets** hold app servers, EKS nodes, RDS.
- **IGW is horizontally scaled and free**; NAT Gateway is the paid, per-AZ piece.
- Same-VPC subnets always talk to each other via the implicit `local` route.

## Summary
- VPC = private network; subnet = AZ-scoped slice; **route table decides public vs private**.
- Public = `0.0.0.0/0 -> IGW`; private outbound = `0.0.0.0/0 -> NAT` (in a public subnet).
- NAT = outbound only, per-AZ, paid. Put ALB/NAT/bastion in public subnets; apps/DBs in private.
