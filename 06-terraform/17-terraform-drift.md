## Terraform Drift

### Question  
What is **Terraform drift**, how do you detect it, and how do you fix it?

### 📝 Short Explanation  
**Drift** is when the **real infrastructure no longer matches the Terraform state/config** — usually because someone changed a resource **outside Terraform** (console click, another tool, an auto-scaling or provider-side change). `terraform plan` is what surfaces it.

## ✅ Answer  
- **Drift = real world ≠ desired state (config/state).** Cause: **out-of-band changes** (manual console edits, scripts, other tools, or cloud-side mutations).
- **Detect** it by running **`terraform plan`** (which refreshes and compares state vs reality) — drift shows as unexpected changes; `terraform plan -refresh-only` shows *only* drift.
- **Fix** it by deciding **which side is the source of truth**: either **re-`apply`** to push config back onto the resource (revert the manual change), **update the code** to match the intended new reality, **`import`** something created outside Terraform, or **`ignore_changes`** for attributes intentionally managed elsewhere.

## 📘 Detailed Explanation  

### 🌊 What drift is and why it happens
Terraform assumes it's the **sole owner** of the resources it manages. Drift appears when reality changes without Terraform:
- Someone edits a resource in the **AWS console** or via CLI/another tool.
- An **auto process** changes it (autoscaling adjusts desired count, a controller mutates tags).
- **Provider/cloud-side** defaults or forced changes.

### 🔍 Detecting drift
```bash
terraform plan                 # refreshes state vs real infra, shows diffs
terraform plan -refresh-only   # show ONLY out-of-band drift (no config changes)
terraform apply -refresh-only  # update state to match reality WITHOUT changing infra
```
- A `plan` that wants to change things you didn't touch in code = **drift**.
- In CI, a **scheduled drift-detection `plan`** (fail/alert on non-empty diff) catches drift early.

### 🔧 Remediating drift (pick the source of truth)
| Situation | Fix |
|---|---|
| Manual change was **wrong** | **`terraform apply`** → Terraform reverts it to match config |
| Manual change was **intended** | **Update the Terraform code** to match, then apply (now it's codified) |
| Resource created **outside** Terraform | **`terraform import`** it into state (+ write matching config) |
| Attribute **legitimately** managed elsewhere (e.g. autoscaling `desired_count`, mutated tags) | **`lifecycle { ignore_changes = [...] }`** so Terraform stops fighting it |

### 🛡️ Preventing drift
- **Least-privilege**: humans don't get write access to prod resources — **all changes go through Terraform** (via CI/PR).
- **`prevent_destroy`** / policy-as-code guardrails on critical resources.
- **Scheduled drift detection** in CI so drift is caught in hours, not during an incident.
- Remote **state + locking** so applies are serialized and state stays authoritative.

### ⚠️ Things they probe
- Drift = **reality ≠ state**, from **out-of-band changes**; **`plan` detects it**.
- Know **`-refresh-only`** (see/record drift without changing infra).
- The fix depends on **who's the source of truth**: re-apply (revert), codify, `import`, or `ignore_changes`.
- Prevention = **no manual changes** (least privilege + everything through Terraform/CI).

---

### 🧠 Real-world Example

> "An engineer widened a security group in the console during an incident. Our nightly **drift-detection `plan`** flagged it the next morning. We decided the change *was* needed, so we **codified it in Terraform** and applied — state and reality realigned, and it was now reviewable. For the RDS `desired`-style fields the app team tuned, we used **`ignore_changes`** so Terraform stopped flapping them."

> Summary:  
> **Drift = real infra diverging from Terraform's state/config due to out-of-band changes.** Detect with **`terraform plan`** (`-refresh-only` for drift-only). Fix by choosing the source of truth: **re-apply to revert, codify the change, `import`, or `ignore_changes`.** Prevent it by forcing **all changes through Terraform** and running **scheduled drift detection**.

### Related
- [Role of the state file](./03-role-of-statefile.md) · [State file management](./05-statefile-management.md)
- [Import blocks & state recovery](./103-import-blocks-and-state-recovery.md) · [Apply fails halfway](./16-apply-fails-halfway.md)
