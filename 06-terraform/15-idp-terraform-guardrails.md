## IDP Guardrails — Making Terraform Changes Safe (No Accidental Updates/Destroys)

### Question  
In an **Internal Developer Platform (IDP)** where a user commits a change to a repo (say, updating an S3 bucket's lifecycle) and the platform runs Terraform to apply it — how do you ensure the change **only touches the intended resource** and doesn't update or destroy existing infrastructure? And if a **bad change is mistakenly approved**, how do you make sure existing resources aren't harmed?

### Short explanation of the question  
This is the real IDP question hiding behind the convoluted phrasing: your platform turns a Git commit into a `terraform apply`. The interviewer wants your **guardrails** — how you keep a self-service, automated apply from silently mutating or deleting production resources, and what stops a mistakenly-approved change from doing damage. It's **defense in depth**, not one control.

---

### Answer  
I layer guardrails so a change is safe **before**, **during**, and **after** approval:
1. **Plan before apply, always** — the platform runs `terraform plan`, surfaces the exact diff (create/update/**destroy**), and a human approves that plan. No blind applies.
2. **Policy-as-code gates** (OPA/Conftest or Sentinel) — automatically **block** any plan that destroys/replaces protected resources or violates rules, so approval isn't the only line of defense.
3. **`prevent_destroy` + `create_before_destroy`** on critical resources — Terraform itself refuses to delete them.
4. **Least-privilege apply role** — the platform's IAM role can only touch what it's allowed to; even an approved bad plan can't delete what the role can't delete.
5. **Small blast radius** — per-team/per-service **separate state**, modular code, so one change can't cascade across everything.

The key mental model: **approval is one gate, not the only gate.** Policy-as-code + `prevent_destroy` + least privilege catch the human mistake.

---

### Detailed explanation of the answer for readers’ understanding

### 🧠 Why Terraform "appends" safely by default
Terraform is **declarative and stateful**: it diffs your config against **state** and real infra, then changes **only what differs**. Updating an S3 lifecycle rule produces an **in-place update** to that one bucket — it doesn't rewrite unrelated resources. The danger is the small set of changes that force **replace** (destroy + recreate) or **destroy**, so the guardrails focus there.

### 🔒 The guardrail stack (defense in depth)

**1. Mandatory `plan` + human-readable diff**
- The platform runs `terraform plan -out=tfplan` and shows the change summary. Reviewers look specifically for `-/+ destroy and then create` and `- destroy` lines.
- Approve the **saved plan** (`apply tfplan`), so what's applied is exactly what was reviewed (no drift between plan and apply).

**2. Policy-as-code (the automated gate)**
- **OPA/Conftest** or **HashiCorp Sentinel** run against the plan JSON in CI.
- Example rules: *deny if any resource is destroyed*, *deny replace on `aws_db_instance`/`aws_s3_bucket`*, *require tags*, *no public S3*. A violation **fails the pipeline** regardless of who approved.
```rego
# pseudo-OPA: block destroys of protected types
deny[msg] {
  rc := input.resource_changes[_]
  rc.change.actions[_] == "delete"
  protected[rc.type]
  msg := sprintf("blocked: %s would be destroyed", [rc.address])
}
```

**3. Terraform-native `lifecycle` protection**
```hcl
resource "aws_s3_bucket" "data" {
  bucket = "prod-data"
  lifecycle {
    prevent_destroy = true          # apply FAILS if the plan would destroy this
  }
}
```
- `prevent_destroy` = hard stop on deletion of critical resources.
- `create_before_destroy` = if a replace is unavoidable, stand up the new one first (no downtime gap).
- `ignore_changes` = stop out-of-band drift (e.g. tags edited in console) from churning the resource.

**4. Least-privilege apply identity**
- The IDP applies with a **scoped IAM role** (and cross-account `assume_role` per target). If the role has no `s3:DeleteBucket`/`rds:DeleteDBInstance`, an approved-by-mistake destroy simply **can't execute**.
- This is the backstop when both review and policy miss something.

**5. Small blast radius + isolation**
- **Separate state per team/service/environment** (distinct S3 keys), so a change is scoped to one stack.
- **Modules** encapsulate resources; targeted, reviewed inputs instead of one giant root.
- **Remote backend: S3 with versioning + DynamoDB locking** — versioning lets you restore a prior state fast; locking prevents concurrent/racing applies.

### 🚑 "A bad change slipped through — now what?" (recovery)
- **Roll back the change in Git** and re-apply (Git is the source of truth in an IDP/GitOps model) — the previous config re-converges the infra.
- If **state** was corrupted, **restore the previous state version** from S3 versioning; use `terraform state` commands for surgical fixes — never hand-edit.
- If a resource was destroyed but still exists in the cloud, **re-import** it (import blocks) rather than recreate.
- Then post-mortem: tighten the policy rule that should have caught it.

### ⚠️ Things they probe
- "How do you stop a destroy?" → **`prevent_destroy` + policy-as-code denying destroys of protected types + least-privilege role.** Name all three.
- "Approval isn't enough" → agree: humans miss `destroy` lines in a long plan; **automated policy** is the reliable gate.
- "How is the apply exactly what was approved?" → **approve a saved plan file** (`-out`), apply that artifact.

---

### 🧪 Real-World Example

> “Our IDP turned a merged PR into `plan` → policy check → gated `apply`. Prod buckets and RDS had `prevent_destroy`, and an **OPA** rule failed the pipeline on any `delete`/`replace` of protected types. Once, a change would have replaced a bucket — OPA blocked it before a human even looked. The apply role also lacked delete permissions on stateful resources, so the worst case was a failed pipeline, never lost data.”

---

### Key takeaway

> “Make automated applies safe with **defense in depth**: mandatory **plan + approve the saved plan**, **policy-as-code** that blocks destroys/replaces of protected resources, **`prevent_destroy`/`create_before_destroy`**, a **least-privilege apply role**, and **small, isolated state**. Approval is one gate — policy, lifecycle rules, and IAM are the gates that catch the human mistake. Recovery = revert in Git and restore the versioned state.”

### Related
- [State file management](./05-statefile-management.md) — versioning, locking, per-env isolation
- [Import blocks & state recovery](./103-import-blocks-and-state-recovery.md) — recovering after a bad change / lost state
- [`count` vs `for_each` & meta-arguments](./12-count-vs-for_each-and-meta-arguments.md) — `lifecycle` / `prevent_destroy`
- [Multi-account & multi-region providers](./13-multi-account-multi-region-providers.md) — least-privilege `assume_role` per account
