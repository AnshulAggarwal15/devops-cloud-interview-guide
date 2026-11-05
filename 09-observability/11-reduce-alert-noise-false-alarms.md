## You Got Woken Up at 2AM by False Alarms — How Do You Reduce Noise?

### Question  
You got woken up at 2AM by false alarms. What's your strategy to reduce alert noise?

### Short explanation of the question  
This is about **alert quality and on-call health**. The interviewer wants to hear that you alert on **symptoms/SLOs that matter**, not every metric — and that you use techniques like grouping, thresholds with duration, and routing to cut noise.

### Answer  
My strategy: alert on **symptoms that affect users (SLOs)**, not on every raw metric; require alerts to be **actionable**; add **duration/for conditions** and sensible thresholds to avoid flapping; **group and deduplicate** related alerts; and **route by severity** so only true "wake me up" issues page at 2AM.

### Detailed explanation of the answer for readers’ understanding

#### 1. 🎯 Alert on symptoms, not causes
- Page on **user-facing SLO breaches** (error rate, latency budget burn), not on "CPU is 80%".
- If it doesn't affect users and isn't urgent, it shouldn't page.

#### 2. ⏱️ Kill the flapping
- Add a **`for:` duration** so a brief spike doesn't fire (e.g. "error rate > 5% for 5m").
- Use **burn-rate** alerts for SLOs (fast burn = page, slow burn = ticket).
- Set thresholds from real baselines, not guesses.

#### 3. 🔕 Reduce volume
| Technique | Effect |
|---|---|
| **Grouping** (Alertmanager) | One notification for many related alerts |
| **Deduplication** | Same alert isn't sent repeatedly |
| **Inhibition** | Suppress downstream alerts when a parent alert fires (node down → don't page for every pod) |
| **Silences/maintenance windows** | Mute during known deploys/maintenance |

#### 4. 🚦 Route by severity
- **Critical/page** → PagerDuty at any hour (real user impact).
- **Warning/ticket** → Slack/email, handled in business hours.
- Only genuine, actionable, user-impacting alerts should wake someone.

#### 5. 🔁 Review regularly
- Track noisy alerts; **tune or delete** ones that are consistently false.
- Every page should be **actionable** and have a runbook.

---

### 🧠 Real-World Example

> On-call was drowning in 2AM pages — mostly transient CPU spikes and duplicate alerts from one root cause. We switched to **SLO burn-rate** alerts, added **`for:` durations**, enabled Alertmanager **grouping/inhibition** (node-down suppressed its pods' alerts), and routed **warnings to Slack, only criticals to PagerDuty**. Overnight pages dropped massively and each remaining one was real.

### Key takeaway

> “Reduce noise by alerting on **user-impacting SLOs**, requiring alerts to be **actionable**, adding **`for:` durations/burn-rate** to stop flapping, using **grouping/dedup/inhibition**, and **routing by severity** so only true emergencies page. Review and prune noisy alerts continuously.”
