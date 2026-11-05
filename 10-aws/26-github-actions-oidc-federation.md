<!-- #genai: AWS interview note - OIDC federation for external systems (GitHub Actions, Databricks) assuming AWS IAM roles; trust policy + AssumeRoleWithWebIdentity + JWKS validation -->

## Question
How does an **external system** (GitHub Actions, or a SaaS like Databricks) authenticate to an AWS account and assume an IAM role? "A role is correct, but a role alone won't work" — what accompanies the role, and how does the integration actually work and get validated?

### Short Explanation
Inside AWS, an EC2 uses an **instance profile** and an EKS pod uses **IRSA** — both are AWS-native. An **external** system has no AWS identity, so a role by itself is useless: there's no way to prove *who* is assuming it. The missing piece is **identity federation** — a **trust relationship** between AWS and an **external OIDC (or SAML) identity provider**, plus **`sts:AssumeRoleWithWebIdentity`** that exchanges a signed **JWT** for temporary AWS credentials. **No long-lived access keys.**

## Answer
- The role needs a **trust policy** that trusts an **external identity provider**, not just a permission policy.
- You register the provider's **OIDC identity provider** in IAM (issuer URL + audience), then write the role's **trust policy** to allow `sts:AssumeRoleWithWebIdentity` **only** for specific subjects (e.g. a repo/branch, or a workload identity).
- At runtime the external system presents a **signed JWT**; **STS validates** the signature against the provider's public keys (**JWKS**) and checks `iss` / `aud` / `sub` against the trust policy, then returns **temporary credentials**.

## Detailed Explanation

### Why "role alone won't work"
A permission policy says *what* the role can do. But AWS must first answer *who is allowed to become this role?* For an internal caller that's an instance profile/IRSA. For an external caller, AWS has no idea who they are — so you must **federate an external identity** and encode "who may assume me" in the **trust policy**. That trust policy + the OIDC provider is "what accompanies the role."

### GitHub Actions → AWS via OIDC (the modern, keyless way)
```text
1. Workflow requests an OIDC token from GitHub  → GitHub signs a JWT
     (claims: iss=token.actions.githubusercontent.com,
              aud=sts.amazonaws.com,
              sub=repo:my-org/my-repo:ref:refs/heads/main)
2. Workflow calls sts:AssumeRoleWithWebIdentity, presenting the JWT
3. STS fetches GitHub's public keys (JWKS) via the OIDC provider you registered,
   verifies the signature, and checks iss / aud / sub against the role's trust policy
4. STS returns TEMPORARY credentials → the job calls AWS
```

**What you configure on the AWS side:**
1. An **IAM OIDC identity provider** for `https://token.actions.githubusercontent.com` (audience `sts.amazonaws.com`).
2. An **IAM role** with a **trust policy** scoped by `sub` so only your repo/branch/environment can assume it:
```json
{
  "Effect": "Allow",
  "Principal": { "Federated": "arn:aws:iam::123456789012:oidc-provider/token.actions.githubusercontent.com" },
  "Action": "sts:AssumeRoleWithWebIdentity",
  "Condition": {
    "StringEquals": { "token.actions.githubusercontent.com:aud": "sts.amazonaws.com" },
    "StringLike":   { "token.actions.githubusercontent.com:sub": "repo:my-org/my-repo:ref:refs/heads/main" }
  }
}
```
On the workflow side you just need `permissions: id-token: write` and the `aws-actions/configure-aws-credentials` step with the role ARN — **no stored secrets**.

### How it's validated (the part they probe)
- The JWT is **signed by the provider's private key**. STS retrieves the provider's **public keys from its JWKS endpoint** (discovered via `/.well-known/openid-configuration`) and verifies the signature — so a forged token fails.
- STS then enforces the trust-policy **conditions**: `iss` (issuer), `aud` (audience), and especially **`sub`** (which repo/branch/env). Scoping `sub` is what stops *any* GitHub repo in the world from assuming your role.

### The same pattern for other external systems (e.g. Databricks)
Databricks lives in its **own ecosystem** — no instance profile. Cross-ecosystem access uses the same idea:
- **OIDC/SAML federation** where supported (Databricks presents a token; AWS role trusts that IdP), or
- **`sts:AssumeRole`** with an external ID / cross-account trust for the Databricks-managed role, or
- Databricks **instance profiles** for clusters running in *your* AWS account.
The principle is identical: **federate an identity, scope the trust policy, exchange for temporary STS credentials** — never embed static keys.

### IRSA is the same mechanism, inside EKS
IRSA is literally OIDC federation where the **provider is your EKS cluster** and the **subject is a Kubernetes ServiceAccount** (`system:serviceaccount:ns:name`). Same `AssumeRoleWithWebIdentity` flow — see the IRSA note.

### Things they probe
- "What accompanies the role?" → **an OIDC/SAML identity provider + a scoped trust policy**, enabling `AssumeRoleWithWebIdentity`.
- "How is it validated?" → **JWT signature via the provider's JWKS + `iss`/`aud`/`sub` conditions**.
- Always **scope `sub`** — an unscoped GitHub trust policy is a critical misconfiguration.

## Summary
- External systems can't use instance profiles/IRSA, so a role needs a **trust relationship with an external OIDC/SAML provider**, not just permissions.
- **GitHub → AWS**: register the **OIDC provider**, create a **role whose trust policy scopes `sub` to your repo/branch**, and the workflow calls **`AssumeRoleWithWebIdentity`** for **temporary, keyless** credentials.
- Validation = **JWT signature checked against the provider's JWKS** + `iss`/`aud`/`sub` conditions. **IRSA is the same pattern with the EKS cluster as the provider.**

## Related
- [IRSA — how an EKS pod gets AWS credentials](./104-irsa.md)
- [AWS STS](./18-aws-sts.md) · [Trust policy](./19-trust-policy.md)
- [IAM users, roles & policies](./103-iam-users-roles-policies.md)
- [Authentication vs Authorization framework](./29-authentication-vs-authorization.md) — OIDC federation is **authN**; the role's trust + permission policy is **authZ**
- [CI/CD from scratch + GitOps deep dive](../05-cicd/17-cicd-from-scratch-gitops-deep-dive.md) — where this OIDC/ECR auth fits in the pipeline
