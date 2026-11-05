## What Happens When You Run `terraform init`?

### Question

What happens when you run `terraform init`?

### Short explanation of the question

`init` is the **first command** you run in any Terraform working directory. The interviewer wants to know you understand it **prepares the working directory** — backend, providers, and modules — and that it's **safe and idempotent** (it doesn't create infrastructure).

---

### Answer

`terraform init` **initializes the working directory**: it configures the **backend** (where state lives), downloads and installs the required **provider plugins**, downloads any **modules**, and writes the **dependency lock file**. It creates **no infrastructure** and is safe to re-run.

---

### Detailed explanation of the answer for readers’ understanding

### 🔧 What it actually does


| Step                 | What happens                                                                                                                                   |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| **Backend init**     | Reads the `backend` block, connects to remote state (e.g. S3 + DynamoDB lock); offers to **migrate** existing state if the backend changed     |
| **Provider install** | Resolves `required_providers`, downloads plugins from the registry into `.terraform/`, and records exact versions in `**.terraform.lock.hcl`** |
| **Module install**   | Downloads child modules (local path, Git, registry) into `.terraform/modules/`                                                                 |
| **Plugin cache**     | Populates `.terraform/` so `plan`/`apply` can run                                                                                              |


### 📄 Artifacts it creates

- `**.terraform/`** directory — providers, modules, and backend metadata (git-ignored).
- `**.terraform.lock.hcl**` — the **dependency lock file** pinning provider versions/checksums (this one **is** committed to Git for reproducible builds).

### 🔁 Key properties

- **Idempotent & safe** — it never touches real infrastructure; re-running just re-syncs plugins/modules.
- Run it again when you **add a provider/module**, **change the backend**, or **clone the repo fresh**.
- Useful flags: `-upgrade` (bump providers/modules to newest allowed versions), `-reconfigure` / `-migrate-state` (backend changes), `-backend-config=...` (inject backend settings).

### 🧭 Where it sits in the workflow

```text
terraform init      # prepare dir: backend + providers + modules
terraform validate  # check syntax/config
terraform plan      # preview changes (config vs state vs real)
terraform apply     # make it real, update state
```

### ⚠️ Things they probe

- "Does `init` create resources?" → **No.** Only `apply` does.
- Commit `**.terraform.lock.hcl`** (reproducibility), git-ignore `**.terraform/**`.
- If you change the backend, `init` prompts to **migrate state** — read it carefully in prod.

---

### 🧪 Real-World Example

> “In CI, the pipeline runs `terraform init` with a `-backend-config` pointing to the per-environment S3 key, restores the provider cache, then runs `plan`. Because we commit `.terraform.lock.hcl`, every runner installs the exact same provider versions — no 'works on my machine' drift.”

---

### Key takeaway

> “`terraform init` **prepares the working directory** — sets up the **backend**, installs **providers** and **modules**, and writes the **lock file**. It's **safe, idempotent, and creates no infrastructure**; re-run it after adding providers/modules or changing the backend.”

### Related

- [Role of the state file](./03-role-of-statefile.md)
- [State file management](./05-statefile-management.md)

