## Question  
Explain a recent challenge that you faced with AWS and how did you solve it.

### 📝 Short Explanation  
An open-ended experience question. The interviewer is checking whether you can tell a **structured incident story**: what broke, how you investigated with evidence, how you fixed it, and what you changed to prevent it. Use a symptom → investigate → root cause → fix → prevention flow.

## ✅ Answer  
I pick one concrete incident and walk through it: the **symptom**, the **evidence** I gathered (CloudWatch/CloudTrail/logs), the **root cause**, the **fix**, and the **prevention**. A strong example is intermittent 5xx errors traced to unhealthy targets behind an ALB.

### 📘 Detailed Explanation  

Use a clear framework so the story lands:

```text
symptoms → scope → recent changes → evidence → root cause → fix → prevention
```

#### 🧩 Example story: intermittent 502s from the ALB

**1. Symptom** — Users saw sporadic `502 Bad Gateway`; not every request.

**2. Scope** — Only one service, only during traffic peaks. Other services fine → not a network-wide issue.

**3. Recent changes** — A new deployment had shipped that morning (a clue, but I still verified with evidence).

**4. Evidence**
- **ALB target group** showed targets flapping **healthy/unhealthy**.
- **CloudWatch** showed memory climbing until the app was OOM-killed and restarted.
- App logs confirmed restarts aligned with the 502 spikes.

**5. Root cause** — A memory leak in the new build; under peak load pods hit the memory limit, failed health checks, and the ALB briefly had no healthy target → 502.

**6. Fix** — Rolled back the bad build immediately to stop the bleeding, then fixed the leak and re-released.

**7. Prevention** — Added a **memory CloudWatch alarm**, tuned health-check thresholds, and added a load test to the pipeline.

---

### 🧠 Why this answer works
- It's **specific** and evidence-driven, not "I googled it."
- It shows you **mitigate first** (rollback) then fix the **root cause**.
- It ends with **prevention**, which is what senior interviewers listen for.

> Summary:  
> Tell **one real incident** using symptom → evidence → root cause → fix → prevention. Show that you led with data (CloudWatch/CloudTrail/logs), mitigated quickly, and closed the loop so it can't recur.
