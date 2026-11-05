## EC2 Instance Terminated Unexpectedly — Troubleshooting

### Question  
An EC2 instance got terminated unexpectedly. How will you troubleshoot?

### Short explanation of the question  
The interviewer wants to see whether you know **where AWS records the "who and why"** of a termination, and that you understand the common automated causes (Auto Scaling, Spot reclamation, failed health checks) versus a human action.

### Answer  
I start with **CloudTrail** to find who or what issued the `TerminateInstances` call, then check whether it was **Auto Scaling**, a **Spot interruption**, or a **manual/API action**. From there I confirm the root cause and add guardrails like termination protection.

### Detailed explanation of the answer

#### 1. 🔎 Check CloudTrail — who called Terminate?
```bash
# Look for the TerminateInstances event
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=TerminateInstances
```
The event shows the **identity** (IAM user/role or an AWS service) and the **source**. This one call usually tells you the whole story:
- A **person/role** → manual or automation using their credentials.
- **autoscaling.amazonaws.com** → an Auto Scaling action.
- **ec2.amazonaws.com** with a Spot reason → Spot reclamation.

#### 2. 📉 Was it Auto Scaling?
- Check the **Auto Scaling group activity history**. A scale-in event or a **failed health check** (EC2 or ELB) makes ASG terminate and replace the instance.
- Fix: tune health checks, cooldowns, or min size.

#### 3. 💸 Was it a Spot Instance?
- Spot instances can be **reclaimed with a 2-minute warning** when capacity is needed or price changes.
- Fix: use On-Demand/Reserved for critical workloads, or diversify instance types / use capacity rebalancing.

#### 4. 🧾 Check the state transition reason
```bash
aws ec2 describe-instances --instance-ids <id> \
  --query 'Reservations[].Instances[].StateTransitionReason'
```
This often states the reason directly (e.g. "User initiated", "Server.SpotInstanceTermination").

---

### 🧠 Real-world Example

> An instance kept disappearing every night. CloudTrail showed the caller was **autoscaling.amazonaws.com** — a scheduled **scale-in** action was removing it. We had put a stateful workload on an ASG-managed instance. Fix: moved it off the ASG and enabled **termination protection**.

### Key takeaway

> "CloudTrail answers *who terminated it*, and the *state transition reason* / ASG activity answers *why*. Most surprise terminations are Auto Scaling scale-in, Spot reclamation, or a manual API call — not random."
