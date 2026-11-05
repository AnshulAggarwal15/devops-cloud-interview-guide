<!-- #genai: AWS interview note - IAM users, roles, policies, policy evaluation, assume-role for the Docusign R2 cloud round -->

## Question
Explain IAM users, roles, and policies — and how AWS decides whether a request is allowed.

### Short Explanation
A **user** is a long-lived identity, a **role** is a temporary identity something *assumes*, and a **policy** is the JSON that grants/denies permissions. On every request AWS evaluates all applicable policies with one rule: **explicit deny always wins**.

## Answer

| Term | What it is |
|---|---|
| **User** | A permanent identity (person or app) with long-lived credentials |
| **Role** | A temporary identity assumed via STS; delivers short-lived credentials |
| **Policy** | JSON document of `Allow`/`Deny` statements (`Effect`, `Action`, `Resource`, `Condition`) |
| **Trust policy** | On a role: *who* is allowed to assume it |
| **Permission policy** | *What* the identity can do once assumed |

## Detailed Explanation

### Prefer roles over keys
- Roles give **temporary** credentials (via **STS**), auto-rotated — no long-lived secrets to leak.
- Examples: EC2 **instance profile**, EKS pod via **IRSA**, cross-account access, CI/CD assuming a deploy role.

### A role has two policies
- **Trust policy** = the `Principal` allowed to `sts:AssumeRole` (e.g. an account, a service, an OIDC provider).
- **Permission policy** = the actions/resources granted after assuming.

### Policy evaluation logic (commonly asked)
1. **Default deny** — everything is denied unless something allows it.
2. An **explicit `Allow`** in any applicable policy grants access...
3. ...**unless** an **explicit `Deny`** matches → **deny always wins**, no matter how many allows exist.
4. Boundaries that can also deny: **SCPs** (org level), **permission boundaries**, **session policies**, resource-based policy conditions.

### `sts:AssumeRole` flow
Call `AssumeRole` -> STS returns temporary `AccessKeyId` + `SecretAccessKey` + **SessionToken** -> use them until they expire. Cross-account = role's trust policy names the other account; that account's user needs `sts:AssumeRole` permission.

## Summary
- **User** = long-lived; **Role** = temporary (STS), preferred everywhere.
- Role = **trust policy** (who can assume) + **permission policy** (what it can do).
- Evaluation: default deny -> explicit allow grants -> **explicit deny always overrides**.
