## SLI, SLO, SLA and Error Budgets

### Question  
What are **SLI, SLO, and SLA** — and what's an **error budget**? How do they drive decisions?

### 📝 Short Explanation  
These are the vocabulary of **reliability as a measurable target**. An **SLI** is the *measurement*, an **SLO** is the *internal target* for that measurement, an **SLA** is the *external contract* (with penalties). The **error budget** is what's left over from the SLO — it turns "how reliable?" into a number you can spend.

## ✅ Answer  
- **SLI (Indicator)** — a **metric** of service health: e.g. % of requests under 300 ms, % of successful requests, availability.
- **SLO (Objective)** — the **internal target** for an SLI over a window: e.g. 99.9% of requests succeed per 30 days.
- **SLA (Agreement)** — the **customer-facing contract** with **consequences** (refunds/credits) if missed; usually **looser** than the SLO.
- **Error budget** — `100% − SLO` = the allowed amount of failure. 99.9% ⇒ **0.1% budget** (~43 min/month). You **spend** it on releases/risk; when it's exhausted, you **slow down and stabilize**.

## 📘 Detailed Explanation  

### The three, in order of tightness
```text
SLI  (what you measure)   → success rate, latency, availability
 ↓
SLO  (your internal goal) → 99.9% over 30 days     ← stricter
 ↓
SLA  (external promise)   → 99.5% or we credit you ← looser (buffer!)
```
The **SLO is deliberately stricter than the SLA** so you get alerted and react **before** you breach the contract.

| Term | Who it's for | Teeth |
|---|---|---|
| **SLI** | Engineering | It's just a number |
| **SLO** | Engineering/product | Internal goal; drives alerts & release policy |
| **SLA** | Customer/legal | **Financial/contractual penalty** if breached |

### Good SLIs (the ones interviewers like)
Frame as **good events / valid events**:
- **Availability**: successful requests / total requests.
- **Latency**: % of requests faster than a threshold (e.g. `< 300 ms`).
- **Error rate**: `5xx / total`.
- Pick SLIs that reflect the **user's experience**, not internal noise (CPU% is not an SLI).

### Error budget — the key idea
- 99.9% SLO over 30 days → **0.1%** may fail ≈ **~43 minutes/month** of "down" (or slow/errored).
- **Budget remaining → ship fast**, take risks, do that big migration.
- **Budget exhausted → freeze features**, focus on reliability/hardening until you're back under SLO.
- This resolves the classic dev-vs-ops tension **with data**: reliability becomes a shared budget, not an argument.

### How it drives decisions (say this)
- **Alerting**: alert on **SLO burn rate** (how fast you're eating the budget), not on every CPU blip — this cuts noise dramatically.
- **Release gating**: no budget left ⇒ deployment freeze.
- **Prioritization**: repeated burn on one SLI ⇒ invest engineering there.

### ⚠️ Things they probe
- **SLO stricter than SLA** (and why: buffer to react before penalties).
- An SLI must be **user-centric** and expressed as a **ratio of good/valid events**.
- **Error budget** connects reliability to velocity — name burn-rate alerting.
- 99.9% ≈ **43 min/month**; 99.99% ≈ 4.3 min/month (know a couple of these).

---

### 🧠 Real-world Example

> "Our checkout SLO was **99.9% of requests succeeding under 300 ms over 30 days**; the customer **SLA** was 99.5% with credits. We alerted on **error-budget burn rate** (fast burn = page, slow burn = ticket) instead of raw CPU. During one bad month we burned the budget early, so we **froze features** and spent two weeks on reliability — the SLA was never breached."

> Summary:  
> **SLI = the measurement, SLO = the internal target (stricter), SLA = the external contract (with penalties).** The **error budget (100% − SLO)** is the allowed failure you spend on velocity; when it runs out, you stabilize. Alert on **burn rate**, and keep SLIs **user-centric**.

### Related
- [Monitoring vs observability](./01-monitoring-vs-observability.md) · [Reduce alert noise](./11-reduce-alert-noise-false-alarms.md)
- [Incident management & P1 handling](./14-incident-management-and-p1.md)
