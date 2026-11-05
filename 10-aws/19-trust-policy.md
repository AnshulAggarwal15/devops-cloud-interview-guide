## Question  
What is a trust policy in AWS and why is it used?

### 📝 Short Explanation  
IAM roles have **two** policies, and people often confuse them. The **permission policy** says *what the role can do*; the **trust policy** says *who is allowed to assume the role*. This question is really testing that distinction.

## ✅ Answer  
A **trust policy** is a resource-based policy attached to an **IAM role** that defines **which principals** (users, services, accounts, or federated identities) are **allowed to assume** that role. It's used with **STS AssumeRole** to control who can take on the role's permissions.

### 📘 Detailed Explanation  

#### 🔐 Two policies on every role
| Policy | Answers | Example |
|---|---|---|
| **Permission policy** | *What* can the role do? | "Allow `s3:GetObject`" |
| **Trust policy** | *Who* can assume the role? | "Allow EC2 / account 111… to assume me" |

Without a trust policy naming you, you **cannot** assume the role even if its permissions are wide open.

#### 📄 Example trust policies

**Let EC2 assume the role (instance profile):**
```json
{ "Effect": "Allow",
  "Principal": { "Service": "ec2.amazonaws.com" },
  "Action": "sts:AssumeRole" }
```

**Let another account assume the role (cross-account):**
```json
{ "Effect": "Allow",
  "Principal": { "AWS": "arn:aws:iam::111111111111:root" },
  "Action": "sts:AssumeRole" }
```

**Federated (OIDC, e.g. IRSA for EKS):**
```json
{ "Effect": "Allow",
  "Principal": { "Federated": "arn:aws:iam::111111111111:oidc-provider/..." },
  "Action": "sts:AssumeRoleWithWebIdentity" }
```

#### 🧠 Why it matters
- It's the **gatekeeper** for role assumption — the first check STS makes.
- Enables **cross-account access**, **service roles** (EC2, Lambda), and **federation** (IRSA/SSO) safely.
- You can tighten it with conditions (e.g. require a specific `sts:ExternalId` for third parties).

---

### 🧠 Real-world Example

> A partner needed to assume a role in our account. We wrote a **trust policy** naming their account as principal and required a unique **`ExternalId`** condition to prevent the "confused deputy" problem. The **permission policy** then scoped them to just the two S3 prefixes they needed.

> Summary:  
> **Permission policy = what the role can do. Trust policy = who can assume it.** The trust policy is the gatekeeper STS checks during `AssumeRole`, and it's what makes service roles, cross-account access, and federation possible.
