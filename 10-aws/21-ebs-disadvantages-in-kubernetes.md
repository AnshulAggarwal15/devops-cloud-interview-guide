## Question  
What are the disadvantages of using EBS volumes in Kubernetes?

### 📝 Short Explanation  
EBS is a **block** volume that attaches to **one node in one Availability Zone at a time**. That single fact drives almost every limitation you hit when you use EBS as persistent storage for pods. The interviewer wants to know you understand `ReadWriteOnce` and the AZ constraint.

## ✅ Answer  
The main disadvantages are: EBS is **`ReadWriteOnce`** (only one node can mount it at a time, so pods can't share it), it's **locked to a single AZ** (a pod rescheduled to another AZ can't attach it), and attach/detach adds **scheduling delays**. For shared, multi-AZ access you'd use **EFS** instead.

### 📘 Detailed Explanation  

#### ⚠️ 1. ReadWriteOnce — no sharing
- An EBS volume can be mounted **read-write by only one node**.
- So you **cannot** have multiple pods across different nodes writing to the same EBS volume.
- This blocks scale-out workloads that need shared storage (use **EFS** with `ReadWriteMany`).

#### ⚠️ 2. Single-AZ binding
- An EBS volume lives in **one AZ**. A pod using it can only be scheduled onto a node in **that same AZ**.
- If that AZ has no capacity, or the pod is rescheduled elsewhere, the volume **can't attach** and the pod stays `Pending`.
- You must use **`topology`-aware** provisioning / `WaitForFirstConsumer` so the volume is created in the right AZ.

#### ⚠️ 3. Attach/detach latency
- On failover, EBS must **detach from the old node and attach to the new one**. This takes time and can slow pod rescheduling, sometimes getting stuck (`Multi-Attach error`) if the old node didn't release cleanly.

#### ⚠️ 4. Scaling limits
- One volume ↔ one pod pattern doesn't scale horizontally; each replica needs its **own** volume (fine for StatefulSets, awkward for shared data).

---

### 🧠 Real-world Example

> A StatefulSet pod backed by EBS got rescheduled to a node in a **different AZ** after a node failure and stayed `Pending` — the volume in AZ-a couldn't attach to a node in AZ-b. We fixed scheduling with `WaitForFirstConsumer` and topology-aware storage classes, and for genuinely shared data we moved to **EFS (ReadWriteMany)**.

> Summary:  
> EBS is **single-node (ReadWriteOnce)** and **single-AZ**, so it can't be shared across pods/nodes and constrains scheduling to one AZ, with attach/detach delays on failover. Use **EFS** when you need shared, multi-AZ (`ReadWriteMany`) storage.
