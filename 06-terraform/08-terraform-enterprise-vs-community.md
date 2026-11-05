## Question  
Do you use Terraform Enterprise or the Community (open-source) version?

### 📝 Short Explanation  
This checks whether you know the difference between the **free CLI (Community)**, **Terraform Cloud**, and **Terraform Enterprise**, and can justify the choice based on **team size, governance, and budget**.

## ✅ Answer  
It depends on scale. Most teams start with the **Community (open-source) CLI** plus a remote backend (S3/DynamoDB). As collaboration and governance needs grow, they move to **Terraform Cloud** (managed remote state, runs, VCS integration) or **Terraform Enterprise** (self-hosted, with SSO, policy-as-code/Sentinel, private module registry, audit logging).

### 📘 Detailed Explanation  

| | Community (CLI) | Terraform Cloud | Terraform Enterprise |
|---|---|---|---|
| Cost | Free | Free tier → paid | Paid (licensed) |
| Hosting | Your machine/CI | **SaaS (HashiCorp)** | **Self-hosted** |
| Remote state + locking | DIY (S3+DynamoDB) | Built-in | Built-in |
| Remote runs | ❌ | ✅ | ✅ |
| **Policy as code (Sentinel/OPA)** | ❌ | Paid tiers | ✅ |
| SSO / RBAC / Audit | ❌ | Higher tiers | ✅ |
| Private module registry | ❌ | ✅ | ✅ |

#### 🧭 How to choose
- **Small team / cost-sensitive:** Community CLI + S3 backend with DynamoDB locking. Cheap and effective.
- **Growing team wanting collaboration:** **Terraform Cloud** for managed state, VCS-driven runs, and remote plan/apply.
- **Enterprise with compliance needs:** **Terraform Enterprise** — self-hosted, **Sentinel policies**, SSO, audit logs, air-gapped installs.

#### 💡 Note on OpenTofu
Some teams also evaluate **OpenTofu** (open-source fork) to avoid licensing concerns for the core engine.

---

### 🧠 Real-world Example

> We began with the **open-source CLI** and an S3/DynamoDB backend. As more teams onboarded, we adopted **Terraform Cloud** for VCS-driven runs and a private module registry. A regulated business unit that needed **Sentinel policy enforcement** and self-hosting used **Terraform Enterprise**.

> Summary:  
> **Community CLI** (free, DIY backend) for small teams; **Terraform Cloud** (managed SaaS) for collaboration; **Terraform Enterprise** (self-hosted) for governance — SSO, **Sentinel policy-as-code**, audit, private registry. Choose by **scale and compliance needs**.
