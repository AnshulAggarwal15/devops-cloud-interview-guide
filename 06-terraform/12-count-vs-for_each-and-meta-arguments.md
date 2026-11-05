## `count` vs `for_each` and Terraform Meta-Arguments

### Question

What are the meta-arguments in Terraform? Explain `count` vs `for_each` — including what happens on **deletion** — and which you'd prefer to create many similar resources.

### Short explanation of the question

Meta-arguments are the special arguments Terraform accepts on **any** resource/module block (not provider-specific). The tricky follow-up is the **deletion behavior**: `count` is index-based, so removing a middle element **re-indexes and recreates** the ones after it, while `for_each` is key-based and only touches the one you removed.

---

### Answer

The core meta-arguments are `**count`**, `**for_each**`, `**depends_on**`, `**provider**`, and `**lifecycle**` (with `create_before_destroy`, `prevent_destroy`, `ignore_changes`, `replace_triggered_by`).

- `**count**` creates N instances addressed by **numeric index** (`aws_instance.web[0]`).
- `**for_each`** creates one instance per **map/set key** (`aws_instance.web["api"]`).
- On deletion, `**count` re-indexes** (removing item 2 of 5 shifts 3→2, 4→3… so Terraform destroys/recreates the tail), whereas `**for_each` is stable** — removing a key only destroys that one resource.
- For multiple similar resources I **prefer `for_each`** because it's keyed and stable, so edits don't cause unintended recreations.

---

### Detailed explanation of the answer for readers’ understanding

### 🧩 The meta-arguments


| Meta-argument    | What it does                                                                                   |
| ---------------- | ---------------------------------------------------------------------------------------------- |
| `**count**`      | Create N copies, addressed by **index** `[0..N-1]`                                             |
| `**for_each`**   | Create one per **key** in a map/set, addressed by `["key"]`                                    |
| `**depends_on`** | Force an **explicit ordering** when Terraform can't infer it                                   |
| `**provider`**   | Pick a **non-default provider** (e.g. `provider = aws.useast1`) — key for multi-account/region |
| `**lifecycle`**  | Control create/update/destroy behavior                                                         |


`**lifecycle` sub-arguments (frequently asked):**

- `create_before_destroy` — stand up the replacement **before** destroying the old (avoid downtime).
- `prevent_destroy` — hard stop; `apply` fails if the plan would destroy this resource (protect prod DBs, buckets).
- `ignore_changes` — ignore drift on specific attributes (e.g. tags mutated outside Terraform).
- `replace_triggered_by` — force replacement when a referenced resource/attribute changes.

### 🔢 `count` — index-based

```hcl
resource "aws_instance" "web" {
  count         = 5
  ami           = "ami-123"
  instance_type = "t3.micro"
}
# addresses: web[0], web[1], web[2], web[3], web[4]
```

### 🔑 `for_each` — key-based

```hcl
resource "aws_instance" "web" {
  for_each      = toset(["api", "worker", "cron", "web", "batch"])
  ami           = "ami-123"
  instance_type = "t3.micro"
}
# addresses: web["api"], web["worker"], ...
```

### ⚠️ The deletion trap (the point they probe)

Say you created 5 with `**count**` and delete the **3rd**:

- `count` addresses by position. Removing index 2 makes former index 3 → 2, 4 → 3, etc.
- Terraform sees `web[2]`, `web[3]`, `web[4]` "changed identity" and will **destroy and recreate** the shifted ones. On real infra that's needless churn/downtime.

With `**for_each`**, each resource is tied to a **stable key**. Delete `web["cron"]` and Terraform destroys **only** that instance — the others are untouched.


|                      | `count`                                   | `for_each`                                  |
| -------------------- | ----------------------------------------- | ------------------------------------------- |
| Addressed by         | Numeric **index**                         | **Key** (map/set)                           |
| Delete a middle item | **Re-indexes → recreates the tail**       | Removes **only** that item                  |
| Best for             | Truly identical, order-independent copies | Named/keyed resources you'll edit over time |
| Input                | A number                                  | A map or set                                |


### 🧭 Which to prefer?

For creating 10 EC2s or replicas of a resource you'll manage over time, **prefer `for_each`** with meaningful keys — it's stable under add/remove and avoids surprise recreations. Reach for `count` only for simple, disposable, index-agnostic duplication (or a quick `count = var.enabled ? 1 : 0` on/off toggle).

---

### 🧪 Real-World Example

> “We first provisioned worker nodes with `count`. When we removed one from the middle of the list, the plan wanted to **recreate several instances** because of index shifting. We switched to `for_each` keyed by node name — after that, removing one node only destroyed that node, with zero churn on the rest.”

---

### Key takeaway

> “Meta-args: `**count`, `for_each`, `depends_on`, `provider`, `lifecycle`**. `count` is **index-based** (deleting a middle item re-indexes and recreates the tail); `for_each` is **key-based** and stable. For managing many similar resources, **prefer `for_each`**.”

### Related

- `[for_each` vs `for](./01-for_each-vs-for.md)` — don't confuse the **meta-argument** `for_each` with the **expression** `for`
- [Multi-account & multi-region providers](./13-multi-account-multi-region-providers.md) — the `provider` meta-argument in action

