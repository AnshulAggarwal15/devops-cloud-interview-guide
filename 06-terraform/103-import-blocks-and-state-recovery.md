<!-- #genai: Terraform import blocks + state-file recovery handbook, for Docusign Cloud/DevOps interview prep -->

# Terraform Import Blocks & State-File Recovery

Two closely-related topics that come up as senior scenario questions: how `import` works (especially the
modern 1.5+ **import blocks**), and what to do when a large state file is lost.

## Table of contents
1. Import blocks — what they are
2. `to` and `id` explained
3. How the import flow works
4. Auto-generating config with `-generate-config-out`
5. Old CLI import vs import blocks
6. Caveats
7. Scenario: the 1000-resource state file is deleted (no backup, no versioning)

---

## 1. Import blocks — what they are
**The old way (pre-1.5)** was a CLI command only:
```bash
terraform import aws_instance.web i-0abc123
```
Two pain points: it was **imperative** (one resource per command, nothing in code), and it **only wrote to
state** — you still had to hand-write matching HCL, and any mismatch made the next `plan` want to
destroy/recreate. Awful for hundreds of resources.

**The new way (Terraform 1.5+)** is a **declarative block in your config**:
```hcl
import {
  to = aws_instance.web    # WHERE it should live in your Terraform config
  id = "i-0abc123"         # the REAL resource's provider ID
}
```

## 2. `to` and `id` explained
- **`to`** = the Terraform **address** (resource type + local name) you want this existing resource tracked
  as. After import, it's managed as `aws_instance.web`.
- **`id`** = the **provider-specific identifier** of the *already-existing* real resource. It differs per
  resource type:
  - EC2 instance → `i-0abc123`
  - S3 bucket → the bucket name
  - IAM role → the role name
  - Security group → `sg-0abc123`

## 3. How the import flow works
1. Add the `import` block(s) to a `.tf` file.
2. `terraform plan` → shows "will import X" (plus any config drift).
3. `terraform apply` → the real resource is pulled **into state**. No destroy, no recreate.
4. Once imported, the block is a **no-op** — safe to delete (or leave; it does nothing on future runs).

## 4. Auto-generating config with `-generate-config-out`
The killer feature. For any import block where you *haven't yet written* the resource HCL:
```bash
terraform plan -generate-config-out=generated.tf
```
Terraform reads the live resource and **writes the HCL for you** into `generated.tf`. You no longer
reverse-engineer 30 arguments by hand — you review, clean up (it's verbose and includes provider defaults),
move it into proper files, and apply.

## 5. Old CLI import vs import blocks

| | CLI `terraform import` | Import blocks (1.5+) |
|---|---|---|
| Style | Imperative (per command) | Declarative (in code) |
| Bulk | One at a time | Many at once, scriptable |
| Config | Hand-written | Auto-generated (`-generate-config-out`) |
| Reviewable in PR | No | Yes (lives in Git) |
| Idempotent | N/A | Yes (no-op once imported) |

## 6. Caveats
- Generated config is a **starting point**: verbose, includes defaults, sometimes needs cleanup.
- A few resource types don't support config generation cleanly.
- Still turns days of manual work into hours.

---

## 7. Scenario: state file with 1000+ resources is deleted — no backup, no versioning

**Lead with the insight that wins the question:**
> Losing `terraform.tfstate` does **not** delete the real infrastructure. All 1000+ resources are still
> running in AWS. Terraform has simply *lost its memory* of them. The danger: if you run `terraform apply`
> now, Terraform sees an empty state, believes nothing exists, and tries to **recreate everything** —
> causing duplicates, conflicts, or destructive churn. **This is a recovery exercise, not a rebuild.**

### Step 1 — Contain the blast radius (first)
Freeze all changes. Lock the CI/CD pipeline so no automated `plan`/`apply` runs. Tell the team: nobody
touches this stack. The worst outcome is an accidental `apply` against empty state.

### Step 2 — Triage for any surviving copy (even if "none" was stated)
Recovery is 100x cheaper than re-import, so always check:
- Local **`terraform.tfstate.backup`** — Terraform auto-writes the previous state here after each apply.
- CI runner **workspace caches / archived artifacts** from the last run.
- A **teammate's local copy** or an old machine.
- Any `.backup` object or lifecycle-archived copy in the bucket, even without versioning.

### Step 3 — Rebuild state by importing the live resources
The infra exists, so reconcile state to reality:
- **`terraform import`** per resource (fine for a few, brutal for 1000).
- **Import blocks (1.5+)** with `-generate-config-out` — import many declaratively and generate config in
  bulk. Generate the blocks from a script that lists resources via the AWS CLI/API.
- **Tooling to scale it:** **Terraformer** (reverse-engineers existing cloud resources into config + state),
  **aws2tf**, or **Former2**.

### Step 4 — Prove it's correct
Iterate `terraform plan` until it reports **"No changes. Your infrastructure matches the configuration."**
That's the definitive signal state now matches both reality and code. Fix drift (tags, defaults) as you go.

### Step 5 — Post-mortem so it never recurs
- **Remote backend: S3 with versioning enabled + DynamoDB locking.** Versioning alone would have made this a
  2-minute restore.
- Restrict who/what can delete state (least-privilege IAM, bucket policies, MFA-delete).
- `prevent_destroy` on critical resources; regular automated state backups; separate state per environment
  to shrink blast radius.

**One-liner that lands it:**
> "State loss isn't a disaster — the resources are intact, so I freeze changes, recover any backup,
> otherwise re-import the resources (import blocks / Terraformer) until `plan` is clean, then make state
> remote, versioned, and locked so it can't happen again."
