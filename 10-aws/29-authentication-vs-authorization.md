<!-- #genai: the authentication vs authorization framework - a universal lens for "how is this step secured?" questions across IAM, K8s RBAC, OIDC/JWT, GitHub, mTLS -->

## Authentication vs Authorization — the Universal Security Framework

### Question
Whenever an interviewer asks "how is this step secured?" / "how does authentication *and* authorization happen here?" — what's the mental framework to answer **any** such question consistently?

### Short Explanation
Almost every "how is it secure?" question decomposes into **two separate questions**, always in this order:
1. **Authentication (authN) — "Who are you?"** Prove identity.
2. **Authorization (authZ) — "What are you allowed to do?"** Check permissions.

If you answer **both, in order, and name the mechanism for each**, you sound senior and you never freeze. This is the lens that makes CI/CD drill-downs ("how does Jenkins get access to GitHub? to ECR? to the cluster?") easy: each hop is just *authN then authZ*.

## Answer
- **AuthN = identity.** *Who is making this request, and how do we trust that?* (password, key, certificate, **signed token/JWT**, OIDC federation, IAM principal.)
- **AuthZ = permission.** *Given who you are, what may you do?* (IAM policy, **RBAC**, scopes, resource policy, group membership.)
- They are **independent and sequential**: you authenticate **first** (establish identity), then authorize (evaluate what that identity can do). A system can authenticate you and still **deny** you (authN success, authZ fail = the classic `403 Forbidden`), vs `401 Unauthorized` = authN failed.

## Detailed Explanation

### The two questions, side by side
| | Authentication (authN) | Authorization (authZ) |
|---|---|---|
| Answers | **Who are you?** | **What can you do?** |
| Establishes | Identity | Permissions |
| Fails with | **401 Unauthorized** | **403 Forbidden** |
| Mechanisms | Passwords, API keys, **X.509 certs / mTLS**, **JWT/OIDC tokens**, SAML, IAM principals, SSH keys | **IAM policies**, **K8s RBAC**, OAuth **scopes**, resource/bucket policies, ACLs, group membership |
| Order | **First** | **Second** |

### Apply it to any system (same lens, different mechanisms)
| System | Authentication (who) | Authorization (what) |
|---|---|---|
| **AWS** | IAM principal / **STS temp credentials** / signed request (SigV4) | **IAM policy** evaluation (identity + resource + SCP; explicit deny wins) |
| **Kubernetes API** | client cert, OIDC token, or (EKS) **IAM → aws-auth/access entry** | **RBAC** (Role/ClusterRole + bindings) |
| **EKS pod → AWS** | **IRSA**: SA token → `AssumeRoleWithWebIdentity` | the assumed **IAM role's** policy |
| **GitHub Actions → AWS** | **OIDC** JWT verified via JWKS | role's **trust policy** (`sub`) + permission policy |
| **API request (JWT)** | signature verified against **JWKS** (`iss`/`aud`) | token **scopes/claims/roles** |
| **TLS / mTLS** | certificate proves server (and client) identity | app-level rules on top |

### Why the order matters (say this)
> "You can't decide *what* someone is allowed to do until you know *who* they are." AuthN produces a **trusted identity**; authZ is a **policy decision** about that identity. Mixing them up is the most common conceptual slip — e.g. "we use a security group" answers *neither* cleanly (that's network reachability, not identity or permission).

### The drill-down cheat: every component hop = authN + authZ
When an interviewer walks each step of a pipeline and asks "how is this secured?", answer each hop the same way:
- **Jenkins → GitHub:** authN = a **GitHub App / deploy key / PAT**; authZ = that credential's **repo scope**.
- **Jenkins → ECR:** authN = **IAM role via OIDC/instance profile** (SigV4); authZ = **ECR permissions** (`ecr:PutImage`).
- **Argo CD → Git:** authN = **deploy key / token**; authZ = read scope on the manifests repo.
- **Argo CD → cluster:** authN = its **ServiceAccount token**; authZ = **RBAC** on what it can apply.
- **Pod → AWS:** authN = **IRSA** token; authZ = the **IAM role** policy.

Same two words, every time. That consistency is what the interviewer is testing.

### Things they probe
- **401 vs 403**: 401 = *not authenticated*; 403 = *authenticated but not authorized*. Getting this right instantly signals you understand the split.
- Name a **mechanism** for each half, don't just say "we secure it."
- **Least privilege** lives on the **authZ** side; **no long-lived credentials / short-lived tokens** lives on the **authN** side.

## Summary
- Break every "how is it secure?" into **authN ("who are you?") then authZ ("what can you do?")** — in that order.
- **AuthN** = identity (tokens/certs/keys/OIDC/IAM principal); **AuthZ** = permissions (IAM policy / RBAC / scopes).
- **401 = authN failed, 403 = authZ failed.**
- For pipeline drill-downs, treat **each component hop as its own authN + authZ pair** and name the mechanism for both — that's the framework that keeps you from freezing.

## Related
- [IAM users, roles & policies](./103-iam-users-roles-policies.md) (authZ: policy evaluation)
- [GitHub/OIDC federation](./26-github-actions-oidc-federation.md) · [IRSA](./104-irsa.md) (authN via federation)
- [Kubernetes RBAC](../08-kubernetes/36-rbac.md) (authZ in the cluster)
- [CI/CD from scratch & GitOps deep dive](../05-cicd/17-cicd-from-scratch-gitops-deep-dive.md) (the framework applied to every hop)
