## Incident Management & Handling a P1

### Question  
How do you handle a **P1 (critical) production incident**? Walk through your process.

### 📝 Short Explanation  
This tests whether you can run an incident **calmly and by process**, not just debug. The senior signal: **mitigate first (restore service), communicate continuously, assign clear roles, then find root cause — and run a blameless postmortem.** Restoring the customer comes before understanding the bug.

## ✅ Answer  
- **Declare & classify** the incident (severity), and **assign roles** (Incident Commander, comms, ops).
- **Mitigate first** — stop the bleeding (roll back, scale out, failover, feature-flag off) **before** deep root-cause analysis. Restoring service beats being right.
- **Communicate** on a cadence to stakeholders/status page.
- **Stabilize → verify recovery (against SLOs)**, then **root cause** and a **blameless postmortem** with action items.

## 📘 Detailed Explanation  

### Severity levels (know the shape)
| Sev | Meaning | Response |
|---|---|---|
| **P1/SEV1** | Major outage / data loss / customer-wide impact | All-hands, page on-call + IC, exec/status-page comms |
| **P2/SEV2** | Significant degradation, partial impact | On-call owns, urgent |
| **P3/SEV3** | Minor/limited, workaround exists | Normal working hours |

### The flow (say it in this order)
```text
Detect (alert/SLO burn) → Declare + classify → Assign roles
   → MITIGATE (rollback/scale/failover/flag)  ← restore service FIRST
   → Communicate on a cadence
   → Confirm recovery (SLIs back to normal)
   → Root cause → Blameless postmortem → Action items
```

1. **Detect & declare.** Triggered by an alert (ideally **SLO burn-rate**), a page, or a report. Declare it so everyone knows it's a live incident.
2. **Roles (for anything sizable):**
   - **Incident Commander (IC)** — coordinates, makes decisions, owns the incident (not necessarily the deepest expert).
   - **Ops/Resolver(s)** — hands on keyboard, investigate/fix.
   - **Comms/Scribe** — updates stakeholders + status page, keeps a timeline.
3. **Mitigate before diagnose.** The goal is **restore service**, not find the bug yet. Fast levers: **roll back the last deploy** (usually the #1 suspect), **scale out**, **failover** to healthy AZ/replica, **feature-flag off** the bad path, shed load. *"I stop the bleeding first — root cause can wait until customers are served."*
4. **Communicate on a cadence.** Regular updates ("investigating / identified / monitoring / resolved") to stakeholders and a **status page**. Silence is worse than bad news.
5. **Confirm recovery.** Verify **SLIs are back to normal** (error rate, latency, availability) — not just "looks fine."
6. **Root cause + blameless postmortem.** Once stable, do the real RCA. Write a **blameless** postmortem: timeline, impact, what happened, **why (systemic, not "who")**, and concrete **action items** with owners. Track **MTTR/MTTD** to improve.

### The mindset that reads as senior
- **Mitigation ≠ resolution.** Rolling back restores service (mitigation); the fix comes later.
- **Blameless.** Blame hides information; you want people to speak freely so the *system* gets safer.
- Tie detection to **SLOs/error budgets** — a P1 often *is* a rapid budget burn.
- Debug *within* the incident using the evidence-driven order (see the debug-order playbook), but never let deep debugging block mitigation.

### ⚠️ Things they probe
- **"Mitigate or root-cause first?"** → **Mitigate.** Restore the customer, then investigate.
- Name the **roles** (IC especially) — shows you've run real incidents.
- **Blameless postmortem + action items** — the difference between reacting and improving.
- The most common first mitigation: **roll back the recent change**.

---

### 🧠 Real-world Example

> "A release spiked 5xx and burned our error budget fast. We declared a **P1**, I took **IC**, one engineer investigated while another handled **comms/status page**. First move was **mitigation — rolled back the deploy**, errors dropped in minutes. Once SLIs were normal we did RCA (a bad DB migration), wrote a **blameless postmortem**, and added a migration check to CI so it couldn't recur."

> Summary:  
> Run it by process: **declare + classify + assign roles (IC/ops/comms) → mitigate first (rollback/scale/failover/flag) → communicate on a cadence → confirm recovery vs SLOs → root cause → blameless postmortem with action items.** Restoring service always comes before finding the bug.

### Related
- [SLI/SLO/SLA & error budgets](./13-sli-slo-sla-error-budgets.md) · [Reduce alert noise](./11-reduce-alert-noise-false-alarms.md)
- [Debug-order playbook](../docusign-debug-order-playbook.md)
