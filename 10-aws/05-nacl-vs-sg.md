## Question  
Explain NACL vs Security Group and which one do you use in your organization?

### 📝 Short Explanation  
Both are virtual firewalls in a VPC, but they operate at **different levels** and behave differently — one is **stateful** (Security Group) and the other is **stateless** (NACL). Interviewers love the stateful vs stateless distinction.

## ✅ Answer  
- A **Security Group** protects an **instance/ENI**, is **stateful**, and supports **allow rules only**.
- A **NACL** protects a **subnet**, is **stateless**, and supports **both allow and deny** rules evaluated in number order.

In my organization we rely on **Security Groups** for almost everything, and use **NACLs** only when we need an explicit **deny** at the subnet edge (like blocking a bad IP range).

### 📘 Detailed Explanation  

| | Security Group | NACL |
|---|---|---|
| Level | Instance / ENI | Subnet |
| State | **Stateful** (return traffic auto-allowed) | **Stateless** (must allow return traffic) |
| Rules | **Allow only** | **Allow and Deny** |
| Evaluation | All rules evaluated; if any allows → allowed | Numbered order, **first match wins** |
| Default | Deny all inbound, allow all outbound | Default NACL allows all; custom NACL denies all |

#### 🔁 Stateful vs stateless (the point they test)
- **Security Group:** if you allow inbound `443`, the response is automatically allowed out. You don't write a return rule.
- **NACL:** if you allow inbound `443`, you **must also** allow the **outbound ephemeral port range** (`1024–65535`) for the reply, or the connection breaks. This trips people up constantly.

#### 🔢 Order of evaluation
- **SG:** no deny rules exist; all allows are OR-ed together.
- **NACL:** rules are numbered and evaluated low → high, stopping at the **first match**. A low-numbered `DENY` beats a higher-numbered `ALLOW`.

---

### 🧠 Real-world Example

> To block a malicious IP range hitting our public subnet, a Security Group couldn't help — SGs can't deny. I added a low-numbered **DENY rule in the subnet's NACL**, which stopped that range for every instance in the subnet at once.

> Summary:  
> **SG = instance level, stateful, allow-only, all rules evaluated.** **NACL = subnet level, stateless, allow+deny, first numbered match wins.** Day to day I use **Security Groups**; I reach for **NACLs** only for subnet-wide explicit denies.
