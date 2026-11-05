## Auto Scaling Group Not Launching EC2 — Troubleshooting

### Question  
An Auto Scaling Group is not launching EC2 instances. What can be the issue?

### Short explanation of the question  
The interviewer wants a checklist of the usual blockers — **service limits, capacity, subnet/AZ config, launch template errors, and IAM/permissions**. The **ASG activity history** is the single best place to start because it prints the failure reason.

### Answer  
I open the **Auto Scaling group's Activity history** first — it states exactly why a launch failed. The common causes are hitting an **EC2/vCPU limit**, **insufficient capacity** in the AZ, a broken **launch template** (bad AMI/instance type/key), **subnet or AZ misconfiguration**, or **IAM permission** issues.

### Detailed explanation of the answer

#### 1. 📜 Read the ASG Activity history
```bash
aws autoscaling describe-scaling-activities \
  --auto-scaling-group-name my-asg
```
The `StatusMessage` usually names the cause directly (e.g. "instance limit exceeded", "no Spot capacity", "AMI not found").

#### 2. 🚧 Work through the common causes

| Cause | Signal | Fix |
|---|---|---|
| **Service limit** | "instance limit exceeded" / vCPU quota | Request a quota increase |
| **Insufficient capacity** | "InsufficientInstanceCapacity" | Add more instance types / AZs; for Spot, diversify |
| **Bad launch template** | "AMI not found", invalid instance type | Fix AMI ID / type / key pair / security group |
| **Subnet / AZ** | No subnets in enabled AZs, or subnet full (no free IPs) | Attach valid subnets across AZs |
| **Max capacity reached** | Desired = Max already | Raise Max size |
| **IAM** | Not authorized to launch | Fix the ASG/service role permissions |
| **Health checks flapping** | Instances launch then terminate | Fix ELB/EC2 health check + grace period |

#### 3. 🔁 Special case: launches then immediately terminates
That's usually a **failing health check** — the app isn't up before the health-check grace period ends. Increase the **health check grace period** or fix startup.

---

### 🧠 Real-world Example

> An ASG stopped adding instances during a traffic spike. Activity history said **"instance limit exceeded"** — we'd hit the account's On-Demand vCPU quota. We requested a **quota increase** and diversified instance types so a single type's capacity couldn't block scaling.

### Key takeaway

> "Start with the **ASG Activity history** — it tells you the reason. Then check **quotas, capacity, launch template, subnets/AZs, max size, and IAM**. If instances launch and die immediately, it's a **health check** problem, not a launch problem."
