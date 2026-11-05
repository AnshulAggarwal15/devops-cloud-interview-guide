## Question  
What is AWS STS and how does it work?

### 📝 Short Explanation  
**STS (Security Token Service)** is the service that issues **temporary, limited-privilege credentials**. It's the engine behind role assumption, cross-account access, and federation. The interviewer wants the concept of *temporary credentials* and the `AssumeRole` flow.

## ✅ Answer  
**AWS STS** issues **short-lived credentials** (access key + secret + session token) that expire after a set time. Instead of long-lived access keys, a principal calls **`AssumeRole`**, STS checks the role's **trust policy**, and returns temporary credentials scoped to that role's permissions.

### 📘 Detailed Explanation  

#### 🔁 The AssumeRole flow
1. A user/service/role calls **`sts:AssumeRole`** on a target role.
2. STS checks the target role's **trust policy** — "is this principal allowed to assume me?"
3. If yes, STS returns **temporary credentials** (with a **session token**) valid for the requested duration (e.g. 15 min–12 hrs).
4. The caller uses those creds to act **as the role**, limited to the role's permission policy.

```bash
aws sts assume-role \
  --role-arn arn:aws:iam::222222222222:role/cross-account-role \
  --role-session-name demo-session
# returns AccessKeyId, SecretAccessKey, SessionToken (temporary)
```

#### 🧰 Key STS operations
| API | Use |
|---|---|
| `AssumeRole` | Cross-account / cross-service role assumption |
| `AssumeRoleWithWebIdentity` | Federation with OIDC (e.g. **IRSA** for EKS pods) |
| `AssumeRoleWithSAML` | Enterprise SSO federation |
| `GetCallerIdentity` | "Who am I?" — great for debugging |

#### ✅ Why temporary credentials matter
- **No long-lived keys** to leak.
- Credentials **auto-expire** → smaller blast radius.
- Basis for **IRSA**, EC2 instance roles, cross-account access, and federation.

---

### 🧠 Real-world Example

> Our EKS pods needed AWS access without static keys. With **IRSA**, the pod's service account token is exchanged via **`AssumeRoleWithWebIdentity`**, and STS hands back **temporary credentials** for the mapped IAM role. Nothing long-lived ever touches the pod.

> Summary:  
> STS = **temporary, expiring credentials**. A principal calls **AssumeRole**, STS validates the **trust policy**, and returns short-lived creds scoped to the role. It powers cross-account access, EC2/instance roles, and federation like IRSA.
