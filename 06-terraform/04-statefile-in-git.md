## Question

Have you considered storing the state file in Git instead of AWS S3 or Azure Blob?

### 📝 Short Explanation

This is a trap-style question. Storing Terraform state in **Git is a bad practice**, and the interviewer wants to hear *why* — no locking, sensitive data exposure, and merge conflicts.

## ✅ Answer

No — storing the state file in **Git is not recommended**. Git has **no state locking**, so concurrent applies corrupt state; state often contains **secrets in plain text** that shouldn't be committed; and Git **merge conflicts** on a machine-generated file are dangerous. Instead I use a **remote backend** like **S3 (with DynamoDB locking)** or **Azure Blob**.

### 📘 Detailed Explanation

#### 🚫 Why Git is the wrong place for state


| Problem                  | Why it matters                                                                                    |
| ------------------------ | ------------------------------------------------------------------------------------------------- |
| **No locking**           | Two people running `apply` at once can **corrupt** the state; Git can't prevent concurrent writes |
| **Sensitive data**       | State stores passwords, keys, tokens in **plain text** → committing to Git leaks them             |
| **Merge conflicts**      | `tfstate` is machine-generated JSON; resolving a merge conflict by hand easily **breaks** it      |
| **No history semantics** | Git tracks text diffs, not resource state; a rollback in Git ≠ safe infra rollback                |


#### ✅ What to use instead — remote backends

Remote backends give you **shared state + locking + encryption**:

- **AWS:** S3 for storage + **DynamoDB** for state locking.
- **Azure:** Blob Storage (built-in lease-based locking).
- **Terraform Cloud/Enterprise:** managed state with locking and access control.

```hcl
terraform {
  backend "s3" {
    bucket         = "my-tf-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "tf-locks"   # locking
    encrypt        = true
  }
}
```

---

### 🧠 Real-world Example

> A team once kept `tfstate` in Git "for versioning." Two engineers applied at the same time, hit a **merge conflict** on the state file, and ended up with duplicated/orphaned resources. We migrated to an **S3 backend with DynamoDB locking**, and the concurrency problems disappeared.

> Summary:  
> Don't store state in Git — **no locking, plaintext secrets, and dangerous merge conflicts**. Use a **remote backend** (S3 + DynamoDB lock, Azure Blob, or Terraform Cloud) with encryption and locking.

