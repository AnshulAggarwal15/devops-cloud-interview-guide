## Question  
How will you disable AWS console access for IAM users?

### 📝 Short Explanation  
This checks that you understand the difference between the **two types of IAM credentials**: a **login profile** (console password) and **access keys** (programmatic/CLI/API). Disabling console access means removing the **login profile**, without necessarily touching programmatic access.

## ✅ Answer  
Console access is controlled by the user's **login profile (password)**. To disable it, I **delete the login profile** (or remove the password). This stops console sign-in while leaving programmatic **access keys** intact — and vice versa. For scale, I enforce it with an **IAM policy / SCP**.

### 📘 Detailed Explanation  

#### 🔑 Two separate credential types
| Credential | Used for | Disable by |
|---|---|---|
| **Login profile (password)** | AWS **Console** sign-in | Deleting the login profile |
| **Access keys** | **CLI / SDK / API** | Deactivating/deleting the access key |

Removing one does **not** remove the other.

#### 🖥️ Disable console access for a user
```bash
# Remove the console password (login profile)
aws iam delete-login-profile --user-name alice
```
Now `alice` can't sign in to the console, but her access keys (if any) still work for the CLI/API.

#### 🔐 Disable programmatic access instead
```bash
aws iam update-access-key --user-name alice \
  --access-key-id AKIA... --status Inactive
```

#### 🏢 Enforce it broadly (best practice)
- Attach an **IAM policy** or an **SCP** (in AWS Organizations) that denies console-oriented actions, or
- Better long term: move users to **federated SSO (IAM Identity Center)** so you don't manage per-user passwords at all, and grant access via roles.

---

### 🧠 Real-world Example

> A service account only needed CLI access but someone had also set a console password. I **deleted its login profile** so it could no longer sign in to the console, keeping just its access key for automation. For human users we moved to **IAM Identity Center (SSO)** and disabled standalone console passwords entirely.

> Summary:  
> Console = **login profile (password)**; programmatic = **access keys** — they're independent. Disable console with `delete-login-profile`. At scale, enforce via **IAM policy/SCP** and prefer **SSO** over per-user passwords.
