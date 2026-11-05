<!-- #genai: AWS interview note - Security Groups vs NACLs for the Docusign R2 cloud round -->

## Question
What is the difference between a Security Group and a Network ACL?

### Short Explanation
Both are virtual firewalls, but at different layers: a **Security Group** protects an **instance/ENI** and is **stateful**; a **NACL** protects a **subnet** and is **stateless**.

## Answer

| | Security Group | NACL |
|---|---|---|
| Attaches to | Instance / ENI | Subnet |
| State | **Stateful** (return traffic auto-allowed) | **Stateless** (must allow return traffic explicitly) |
| Rules | **Allow only** | **Allow and Deny** |
| Evaluation | All rules evaluated; if any allows, it's allowed | Rules processed **in number order**, first match wins |
| Default | Denies all inbound, allows all outbound | Default NACL allows all; custom NACL denies all until you add rules |

## Detailed Explanation

### Stateful vs stateless (the point they test)
- **Security Group (stateful):** if you allow inbound `443`, the response is automatically allowed back out. You don't write a return rule.
- **NACL (stateless):** if you allow inbound `443`, you **also** must allow the **outbound ephemeral port range** (`1024-65535`) for the response, or it breaks. This bites people constantly.

### Order of evaluation
- **SG:** all rules are OR-ed; there are no deny rules, so you allow the minimum you need.
- **NACL:** rules are numbered; AWS evaluates lowest-to-highest and **stops at the first match**. A low-numbered `DENY` beats a higher-numbered `ALLOW`.

### When you'd use a NACL
- SGs handle almost everything day to day.
- Reach for a NACL when you need an explicit **deny** at the subnet edge (e.g. block a bad IP range) — SGs can't deny.

## Summary
- **SG = instance level, stateful, allow-only, all rules evaluated.**
- **NACL = subnet level, stateless, allow+deny, first numbered match wins.**
- Stateless means NACLs need explicit rules for **return traffic** (ephemeral ports `1024-65535`).
