## Question  
Two DevOps engineers attempt to update the state file at the same time. What happens?

### 📝 Short Explanation  
This tests your understanding of **state locking**. With a proper backend, Terraform prevents simultaneous writes; without locking, concurrent applies can **corrupt** the state.

## ✅ Answer  
With a backend that supports **state locking** (like S3 + **DynamoDB**, Azure Blob, or Terraform Cloud), the **first** engineer's `apply` acquires a **lock**, and the **second** is blocked with a "state locked" error until the first finishes. Without locking, both could write at once and **corrupt** the state.

### 📘 Detailed Explanation  

#### 🔒 How locking works
When you run `plan`/`apply`, Terraform **acquires a lock** on the state before making changes and **releases** it when done.
- **S3 backend** uses a **DynamoDB table** to hold the lock.
- **Azure Blob** uses blob leases.
- **Terraform Cloud** locks the workspace.

The second concurrent run sees the lock and refuses to proceed:
```text
Error: Error acquiring the state lock
Lock Info:
  ID:        <uuid>
  Who:       user2@host
  Created:   ...
```

#### 🧨 Without locking (e.g. local state or Git)
Both applies read the same state, make changes, and **overwrite** each other's updates → **state corruption**, orphaned or duplicated resources, and drift that's painful to untangle.

#### 🛠️ If a lock gets stuck
If a run crashes and leaves a stale lock, you can force-unlock **carefully** (only when you're sure no apply is running):
```bash
terraform force-unlock <LOCK_ID>
```

---

### 🧠 Real-world Example

> Two engineers kicked off `apply` on prod within seconds of each other. Because we used **S3 + DynamoDB locking**, the second got a **state lock error** and simply waited. Before we had locking, a similar race had once duplicated a security group and corrupted state — locking ended that class of problem.

> Summary:  
> With **state locking**, the second concurrent apply is **blocked** until the first releases the lock — protecting state integrity. Without locking (local/Git state), simultaneous writes **corrupt** the state. Always use a locking backend; `force-unlock` only for genuinely stale locks.
