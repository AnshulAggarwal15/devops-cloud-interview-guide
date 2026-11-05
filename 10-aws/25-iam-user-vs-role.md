## Question  
What is the difference between an IAM user and an IAM role?

### 📝 Short Explanation  
This tests the core IAM identity concept: a **user** is a permanent identity with **long-term credentials**, while a **role** is an identity with **no permanent credentials** that is **assumed temporarily**. It leads naturally into STS and best practices.

## ✅ Answer  
- An **IAM user** is a **permanent identity** for a person or app, with **long-term credentials** (password and/or access keys).
- An **IAM role** has **no long-term credentials**; it's **assumed** by a trusted principal (a service, user, another account, or federated identity) and hands back **temporary credentials** via STS.

Best practice is to prefer **roles** and avoid long-lived user access keys.

### 📘 Detailed Explanation  

| | IAM User | IAM Role |
|---|---|---|
| Identity type | Permanent | Assumed temporarily |
| Credentials | **Long-term** (password, access keys) | **Temporary** (via STS) |
| Who uses it | A specific person or app | Anyone the **trust policy** allows |
| Trust policy | ❌ No | ✅ Yes (defines who can assume) |
| Typical use | Human login / legacy programmatic | EC2/Lambda service access, cross-account, federation, **IRSA** |
| Security | Keys can leak / go stale | Auto-expiring, smaller blast radius |

#### 🔑 The key idea
- A **user** *is* an identity you log in as.
- A **role** is a set of permissions you **temporarily borrow**. It only works if its **trust policy** names you as an allowed principal, and STS then issues short-lived credentials.

#### ✅ Why roles are preferred
- **No static keys** to leak or rotate.
- Credentials **expire automatically**.
- One role can be assumed by many principals; great for **services** (EC2/Lambda), **cross-account** access, and **federation/SSO**.

---

### 🧠 Real-world Example

> Instead of baking an IAM **user's** access keys into an EC2 app (which we'd have to rotate and could leak), we attached an **IAM role** to the instance. The app automatically received **temporary credentials** from the instance metadata — nothing long-lived on disk. Same pattern powers **IRSA** for EKS pods.

> Summary:  
> **User = permanent identity with long-term credentials.** **Role = no permanent credentials, assumed via a trust policy, giving temporary STS credentials.** Prefer roles (for services, cross-account, federation) over long-lived user access keys.
