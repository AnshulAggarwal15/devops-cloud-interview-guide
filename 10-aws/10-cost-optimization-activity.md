## Question  
Explain a cost optimization activity that you performed in your current organization.

### 📝 Short Explanation  
This is an **experience-based** question. The interviewer wants a real, structured story: how you **found** the waste, **what** you changed, and the **measurable savings** — not a generic list. Frame it as identify → act → measure.

## ✅ Answer  
I used **Cost Explorer and the Cost & Usage Report** to find the biggest spenders, then applied targeted actions: **rightsizing** over-provisioned instances, buying **Savings Plans / Reserved Instances** for steady workloads, deleting **idle/unattached resources**, and moving cold data to **cheaper S3 tiers**. This cut our monthly bill measurably without touching reliability.

### 📘 Detailed Explanation  

#### 1. 🔎 Find where the money goes
- **Cost Explorer** grouped by service, tag, and account.
- **CUR** for line-item detail; **Trusted Advisor** and **Compute Optimizer** for specific recommendations.
- Enforce **cost allocation tags** so spend maps to teams/environments.

#### 2. 💡 Common high-impact actions

| Lever | What I did |
|---|---|
| **Rightsizing** | Downsized EC2/RDS that ran at <20% CPU (Compute Optimizer). |
| **Savings Plans / RIs** | Committed 1-year on steady baseline compute for ~30–50% off. |
| **Idle cleanup** | Deleted unattached **EBS volumes**, old **snapshots**, unused **Elastic IPs**, idle load balancers. |
| **Storage tiering** | **S3 Lifecycle** to move cold data to IA/Glacier; enabled **Intelligent-Tiering**. |
| **Scheduling** | Stopped **non-prod** EC2/RDS nights & weekends. |
| **Autoscaling** | Scale in aggressively; use **Spot** for fault-tolerant workloads. |

#### 3. 📈 Measure and sustain
- Track the before/after in Cost Explorer.
- Set **AWS Budgets** with alerts to prevent regressions.

---

### 🧠 Real-world Example

> Cost Explorer showed EC2 as our top spend. **Compute Optimizer** flagged half the fleet as over-provisioned. We rightsized those, bought a **Compute Savings Plan** on the steady baseline, and scheduled non-prod to stop overnight. Net result: roughly a **35% drop** in the monthly compute bill with no performance impact.

> Summary:  
> **Measure first (Cost Explorer/CUR), then act:** rightsize, commit with Savings Plans/RIs, delete idle resources, tier storage, and schedule non-prod. Always tie it back to a **number** and guard it with Budgets.
