## What Happens When `terraform apply` Fails Halfway?

### Question  
You run `terraform apply`, it creates some resources, then **fails partway through**. What state are you in, and how do you recover?

### Short explanation of the question  
This tests whether you understand that Terraform is **not transactional** — there's **no automatic rollback**. Resources created before the failure **stay created and are recorded in state**; the rest aren't. Recovery is about understanding **what actually got applied**, fixing the cause, and **re-running** (Terraform is designed to converge on re-apply).

---

### Answer  
- Terraform applies resources in **dependency order** and **records each success in state as it goes**. When one resource fails, it **stops** and **does not roll back** the ones already created.
- So you're left with a **partially applied** infrastructure: successful resources exist and are **tracked in state**; the failed one and everything depending on it were **not created**.
- **Recovery:** read the error, fix the root cause (bad config, quota, permissions, dependency), then **re-run `terraform plan`** — it shows exactly what's left to do — and **`apply` again**. Because Terraform is **declarative and idempotent**, it creates only the missing/changed pieces.

---

### Detailed explanation of the answer for readers’ understanding

### 🧠 Why there's no rollback
Terraform is **declarative, not transactional**. It walks the dependency graph and, for **each** resource, makes the change and **immediately writes the result to state**. A failure halts the graph — it doesn't undo prior successes (unlike a DB transaction). This is deliberate: those resources really exist in the cloud, so pretending they don't would be worse.

### 📋 What the state looks like after a partial apply
- Resources created **before** the failure → **exist** and are **in state**. Re-running won't recreate them.
- The **failed** resource → not created, not in state (or tainted, see below).
- Resources **downstream** of the failure → never attempted.
- `terraform plan` after the failure = the **accurate to-do list** of what's left.

### 🔧 Recovery steps
1. **Read the actual error** — quota limit, IAM permission, name collision, invalid value, provider/API error, dependency not ready.
2. **Inspect reality vs state:**
   ```bash
   terraform state list          # what Terraform thinks exists
   terraform plan                # what's still needed / any drift
   ```
3. **Fix the root cause** (config, credentials, quota, a race/`depends_on`).
4. **Re-apply** — Terraform converges: creates the missing resources, leaves existing ones alone.
5. **Edge cases:**
   - A resource created in the cloud but the write-to-state failed (e.g. apply crashed) → you may get a **duplicate** on re-apply or a "already exists" error → **`terraform import`** it into state, then continue.
   - A resource left in a **bad/half-configured** state → **`terraform taint`** (or `-replace`) it so the next apply recreates it cleanly.
   - A crash can leave a **stale state lock** → `terraform force-unlock <ID>` (only when sure nothing's running).

### 🛡️ Prevention / good practice
- Always **`plan` and review** before `apply`; in CI, **apply a saved plan** (`-out`) so apply matches review.
- **Smaller, modular stacks** and **separate state per component/env** → a failure's blast radius is small.
- **Idempotent, dependency-correct** config (`depends_on` where the graph can't infer ordering) avoids race-type failures.
- Watch **provider retries / eventual consistency** (e.g. IAM propagation) — a common "fails then succeeds on re-run" cause.

### ⚠️ Things they probe
- **No automatic rollback** — say this first. Partial apply leaves real resources.
- Re-running is **safe and expected** because Terraform is **idempotent** and converges.
- Know **`import`** (resource exists but not in state) vs **`taint`/`-replace`** (recreate a bad resource) vs **`force-unlock`** (stale lock).
- `plan` after a failure is your **source of truth** for what's left.

---

### 🧠 Real-World Example

> "An apply created a VPC and subnets, then failed on an RDS instance — we'd hit an instance-type quota. Nothing rolled back: the VPC/subnets were live and in state. We requested the quota bump, ran `terraform plan` (which showed only the RDS + its dependents remaining), and re-applied — it picked up exactly where it left off. Once, an EC2 got created but the state write failed on a crash; we `terraform import`ed it so the next apply didn't try to duplicate it."

---

### Key takeaway

> "Terraform is **declarative, not transactional — there's no rollback.** A mid-apply failure leaves the already-created resources **live and in state**; you **read the error, fix the cause, and re-`apply`**, and Terraform **idempotently** finishes the rest. Use **`import`** for resources that exist but aren't in state, **`taint`/`-replace`** for half-broken ones, and **`force-unlock`** for a stale lock. Prevent pain with **plan-review, small modular stacks, and per-env state**."

### Related
- [Role of the state file](./03-role-of-statefile.md) · [State file management](./05-statefile-management.md)
- [Concurrent state update / locking](./06-concurrent-statefile-update.md) · [Import blocks & state recovery](./103-import-blocks-and-state-recovery.md)
- [IDP Terraform guardrails](./15-idp-terraform-guardrails.md)
