# Prep: Experience + DB + Scripting + Fitment Round

**Round shape (from the signal):** starts on **your experience**, then **grills** into it; also **DB**, **scripting/coding**, and **fitment** for the role/team. The interviewer's own advice — *take time to think, be open to feedback, be honest about gaps* — is the playbook. Follow it literally.

---

## 0) How to handle THIS interviewer (most important — it targets your failure mode)
Your recurring weakness (from every transcript) is **rambling/keyword-bluffing when unsure**. This interviewer punishes that and rewards the opposite. So:
- **Pause 2–3 seconds and structure before you speak.** "Let me think for a second." A short silence reads as senior; rambling reads as junior. This is the single biggest behavioral fix.
- **Answer in shape: direct answer → mechanism → example.** Not a wandering preamble.
- **Be honest about gaps — he explicitly invited it.** "I've used X at a usage level, not the internals" is a *strong* answer here. Bluffing a mechanism you don't have is the one thing that will sink you.
- **Treat feedback as a conversation.** If he corrects or nudges, say "that makes sense — so in that case…" and build on it. Don't get defensive; he's testing collaboration/coachability (that's the "fitment" part).
- **Never claim depth you can't defend.** Everything you say about your experience is fair game to grill — only put weight on what you can go 2 layers deep on.

---

## 1) Own your experience (this is where the round lives)
He starts here and drills. For **each** project/bullet on your resume, prepare this **one-pager** (fill the brackets — only you have the content):

```
PROJECT: [name / IDP / whatever]
- Problem it solved:        [why it existed]
- MY specific role:         [what YOU did — not "the team"]
- Architecture (30 sec):    [components + how they connect]
- Tech used:                [tools + WHY each was chosen]
- Hardest challenge:        [what broke / was hard + how you solved it]
- Trade-off you made:       [X over Y, and why]
- What you'd improve:       [shows maturity]
```
Prioritize the **IDP** story — it's on your resume and got drilled hard at Nielsen and Amagi. Also prep the daily-work / Argo CD + Jenkins + GitHub + RDS stories.

**Likely grill questions** (rehearse out loud):
- "Walk me through [project]." → 60–90 sec, architecture-first.
- "Why did you choose [tool] over [alternative]?"
- "What was the hardest bug/incident, and how did you handle it?"
- "What would you do differently?"
- "What part did *you* build vs the team?"

**Honesty scripts (memorize 2–3):**
- *"I've operated X day-to-day, but I haven't gone deep on its internals — here's what I do understand…"*
- *"I haven't done that myself; conceptually I'd approach it as…"*
- *"I don't know that — how do you handle it here?"* (turns a gap into a conversation)

**Reference (your own repo):** [13-project-related](../13-project-related) — typical day, devops experience, team contribution, current-org project.

---

## 2) DB prep (a named focus)
You have **RDS** experience — lead with that, and be ready for both **SQL concepts** and **operational** DB questions.

**Concepts to be able to say in one line each:**
- **ACID** (atomicity, consistency, isolation, durability) — what a transaction guarantees.
- **Indexes** — speed reads, cost writes/space; when they help (WHERE/JOIN/ORDER BY) and hurt.
- **Normalization vs denormalization** — reduce redundancy vs read performance.
- **Joins** — inner/left/right; be ready to write one.
- **`EXPLAIN` / query plan** — how you find a slow query.
- **Transactions & isolation levels** — read committed vs repeatable read; deadlocks.
- **Connection pooling** — why (DB connections are expensive; app should pool / use RDS Proxy).
- **SQL vs NoSQL** — when each.

**Operational / AWS RDS (your ground — know these cold):**
- **Multi-AZ vs read replicas** — Multi-AZ = **failover/HA (synchronous standby, not readable)**; read replicas = **read scaling (async, readable)**. Don't confuse them.
- Backups/PITR, storage autoscaling, "RDS is full" response.
- **Reference:** [111-rds-multiaz-replicas-scaling](../10-aws/111-rds-multiaz-replicas-scaling.md) · [08-rds-storage-full](../10-aws/08-rds-storage-full.md) · [23-day-to-day-database-activities](../10-aws/23-day-to-day-database-activities.md)

**Likely questions:** "How would you debug a slow query?" · "Multi-AZ vs read replica?" · "How do you handle DB migrations/backups?" · "Write a query to get [X]." Be honest if your SQL is rusty — say "I'm stronger on the operational/RDS side than deep query tuning."

---

## 3) Scripting / coding (a named focus)
Likely a small **Bash or Python** task, or "explain a script you wrote." You have real scripts in the repo — know them cold.

**Bash — be ready to write/explain:**
- Log parsing: `grep`/`awk`/`sed`, `tail -f`, counting occurrences.
- File ops: find files older than N days / larger than N MB, delete/rotate logs.
- A loop over a CSV to create users; a service-health check loop.
- **Reference:** [03-linux/01-most-used-linux-commands](../03-linux/01-most-used-linux-commands.md) + your scripts `create_users.sh`, `log_rotation.sh`, `monitor_services.sh` (know what each line does).

**Python — automation angle:**
- **boto3** (list/stop instances, S3), **idempotency** (check before create), **error handling/retries**, log parsing.
- **Reference:** [12-python/04-python-conceptual-interview-questions](../12-python/04-python-conceptual-interview-questions.md) · [12-python/05-python-programming-interview-questions](../12-python/05-python-programming-interview-questions.md)

**If asked to code live:** think out loud, state your approach first, write simple/correct over clever, then mention edge cases. Same rule — a clear "here's how I'd approach it" beats silent flailing.

---

## 4) Fitment / behavioral
He's assessing fit for **his team**. Have crisp answers (STAR: Situation → Task → Action → Result) for:
- **"Tell me about yourself"** (60 sec: years, stack — AWS/Terraform/EKS/Jenkins/Argo CD/Python — and what you focus on).
- **Why this role / why this firm** (finance = reliability, security, controlled change — lean into that).
- **A conflict / disagreement** and how you handled it.
- **A mistake/failure** and what you learned (be real; shows growth).
- **How you handle competing priorities.**
- **How you learn new tech** (honest: "I'm actively deepening Kubernetes right now" is fine and mature).
- **A question for HIM** (about the team, on-call, how they do deployments/change management).

Finance-firm angles to weave in: **change control, approvals, auditability, security/least-privilege, reliability/blast-radius** — you have real material here (IDP guardrails, GitOps rollback, IRSA/least-privilege).

---

## 5) The 6 things to rehearse OUT LOUD before the call
1. Your **IDP walkthrough** (90 sec, architecture-first).
2. **"Tell me about yourself"** (60 sec).
3. **Multi-AZ vs read replica** (one clean sentence).
4. **A Bash one-liner** you can write from memory (e.g., delete logs older than 30 days).
5. **One honesty script** ("I've used it at a usage level, not the internals — here's what I know…").
6. **The pause.** Literally practice: hear question → breathe → "let me think" → structured answer.

> Your edge in this round isn't depth — it's **composure + honesty**. This interviewer told you exactly how to win: think, be honest, be coachable. Do that and your gaps become "areas I'm actively growing," not disqualifiers.
