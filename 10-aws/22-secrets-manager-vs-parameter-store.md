## Question  
What is the difference between AWS Secrets Manager and SSM Parameter Store?

### 📝 Short Explanation  
Both store configuration and secrets, and both encrypt with KMS — so interviewers want to hear the **real differentiators**: automatic **rotation**, **cost**, and when each is the right tool.

## ✅ Answer  
- **Parameter Store** stores config and secrets; standard parameters are **free**, and it's great for plain config values.
- **Secrets Manager** is purpose-built for secrets, with **built-in automatic rotation** (e.g. RDS credentials) and cross-region replication — but it **costs per secret**.

I use **Parameter Store for general config** and **Secrets Manager when I need automatic rotation** of credentials.

### 📘 Detailed Explanation  

| | Parameter Store (SSM) | Secrets Manager |
|---|---|---|
| Purpose | Config + secrets | Secrets first |
| Encryption | KMS (SecureString) | KMS (always) |
| **Rotation** | ❌ Not built in (DIY with Lambda) | ✅ **Built-in** (native RDS/Redshift/DocumentDB) |
| Cost | **Standard tier free** | **Per secret / month + API calls** |
| Cross-region replication | Manual | ✅ Built-in |
| Size limit | 4 KB (std) / 8 KB (advanced) | Up to 64 KB |
| Versioning | Yes | Yes |

#### 🔑 The deciding factor: rotation
- If you need credentials to **rotate automatically** (say, an RDS password every 30 days), **Secrets Manager** does it natively with a managed Lambda.
- Parameter Store can't rotate on its own — you'd build it yourself.

#### 💰 The other factor: cost
- Parameter Store **standard** parameters are free → ideal for lots of non-secret config (feature flags, endpoints, versions).
- Secrets Manager charges **per secret per month**, so don't dump plain config into it.

---

### 🧠 Real-world Example

> Database passwords went into **Secrets Manager** with **automatic 30-day rotation** wired to RDS — no human ever touched them. Application config like log levels, service endpoints, and image tags went into **Parameter Store** (free tier). Best of both, and the bill stayed small.

> Summary:  
> **Parameter Store** = cheap/free config store, no native rotation. **Secrets Manager** = secrets with **built-in automatic rotation** and replication, at a per-secret cost. Pick Secrets Manager when **rotation** matters; Parameter Store for everyday config.
