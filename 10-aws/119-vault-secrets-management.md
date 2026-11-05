<!-- #genai: AWS interview note - HashiCorp Vault, dynamic secrets, auth methods, secret engines, vs Secrets Manager for the cloud round -->

## Question
What is HashiCorp Vault, how does it work, and when would you use it over AWS Secrets Manager?

### Short Explanation
Vault is a **secrets-management and encryption platform**. Interviewers want the concepts — **auth methods, secret engines, dynamic secrets, leases** — and the judgment call: Vault shines for **dynamic, short-lived credentials** and **multi-cloud**; Secrets Manager is simpler if you're all-in on AWS.

## Answer
- Vault **centrally stores, generates, and encrypts secrets**, gating access through **policies** after an app/user authenticates via an **auth method**.
- Its standout feature is **dynamic secrets**: it generates **short-lived, on-demand credentials** (DB users, cloud creds) with a **lease/TTL** and auto-revokes them.
- Use **Vault** for dynamic/short-lived secrets, encryption-as-a-service, and **multi-cloud/on-prem**; use **AWS Secrets Manager** when you want a managed, AWS-native store with built-in rotation and less to operate.

## Detailed Explanation

### Core building blocks
| Concept | Role |
|---|---|
| **Auth method** | How a caller proves identity → gets a token (AppRole, **Kubernetes**, **AWS IAM**, OIDC, LDAP) |
| **Token + policy** | Token carries **policies** (HCL) that allow/deny paths |
| **Secret engine** | Mounted backend that stores or **generates** secrets |
| **Lease / TTL** | Every dynamic secret has a lifetime; renew or it's **revoked** |
| **Seal / unseal** | Vault starts **sealed**; unseal (Shamir keys or auto-unseal via KMS) to decrypt storage |

### Secret engines (what it can do)
- **KV** — static key/value secrets (versioned).
- **Database** — **generates** DB users on demand with a TTL, then drops them.
- **AWS** — issues **temporary IAM credentials/STS** dynamically.
- **PKI** — acts as a CA, issues short-lived TLS certs.
- **Transit** — **encryption as a service** (encrypt/decrypt without exposing keys).

### Dynamic secrets — the killer feature
Instead of one static DB password shared forever, Vault creates a **unique, expiring** DB user per app/session. If leaked, it's useless after the TTL, and you get per-consumer auditability. This is the main reason teams pick Vault.

### Vault on Kubernetes / AWS
- Apps authenticate with the **Kubernetes auth method** (using their ServiceAccount token) or **AWS IAM auth**, then read secrets — no static bootstrap secret.
- Injection via the **Vault Agent sidecar** or **CSI provider**; auto-unseal typically backed by **AWS KMS**.

### Vault vs AWS Secrets Manager
| | Vault | Secrets Manager |
|---|---|---|
| Hosting | Self-managed (or HCP) | Fully managed |
| Dynamic secrets | ✅ Strong (DB/cloud/PKI) | Limited (rotation lambdas) |
| Multi-cloud / on-prem | ✅ Yes | AWS-centric |
| Encryption-as-a-service | ✅ Transit engine | ❌ |
| Ops burden | Higher (unseal, HA, upgrades) | Low |
| Rotation | Dynamic + rotation | Built-in native rotation |

### Things they probe
- **Sealed vs unsealed** and why (keys never sit in plaintext on disk).
- **Leases/TTL + revocation** — how a leaked secret self-expires.
- Don't store the Vault token in code; bootstrap trust via workload identity (K8s SA / IAM).

## Summary
- Vault = central **secrets + encryption** platform: **auth method → policy-scoped token → secret engine**.
- **Dynamic, short-lived secrets** with **leases/TTL** are the differentiator; **Transit** gives encryption-as-a-service.
- Choose **Vault** for dynamic creds, PKI, and **multi-cloud**; **Secrets Manager** for a low-ops, AWS-native store with native rotation.

## Related
- [Secrets Manager vs Parameter Store](./22-secrets-manager-vs-parameter-store.md)
- [Cloud security fundamentals](./118-cloud-security-fundamentals.md)
- [Kubernetes secret types](../08-kubernetes/35-types-of-secrets.md)
