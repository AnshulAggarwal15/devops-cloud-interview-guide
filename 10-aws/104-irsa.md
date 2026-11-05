<!-- #genai: AWS interview note - IRSA (IAM Roles for Service Accounts) explained simply with an analogy, for the Docusign R2 cloud round -->

## Question
How does a pod running in EKS get AWS permissions securely (without baking in access keys)?

### Short Explanation
**IRSA** lets a Kubernetes **ServiceAccount** "wear" an **AWS IAM role**. Pods using that ServiceAccount
automatically get **temporary AWS credentials** — no access keys stored anywhere.

---

## The problem it solves (start here)
A pod needs to call AWS (e.g. read an S3 bucket). How does AWS know *who* the pod is and *what* it's allowed
to do? Three options:
- ❌ **Hard-code access keys** in the pod → long-lived secrets that leak and must be rotated by hand.
- ❌ **Use the node's IAM role** → *every* pod on that node shares the same broad permissions (huge blast
  radius).
- ✅ **IRSA** → each workload gets its **own** IAM role with least privilege, and credentials are
  **temporary and auto-rotated**.

## The simple analogy
Think of it like an **airport**:
- The pod shows a **boarding pass** (a signed token that proves "I am ServiceAccount `orders` in namespace
  `prod`").
- **AWS STS is the gate agent.** It checks the boarding pass against a **trusted issuer** (your cluster's
  OIDC provider) and, if it matches the role's rules, hands the pod a **temporary visitor badge**
  (short-lived AWS credentials) that expires automatically.

No permanent keys change hands — just a verifiable token exchanged for a temporary badge.

## How it's wired (3 setup pieces)
1. **EKS publishes an OIDC provider** — a trusted "identity issuer" URL for the cluster. You register it in
   IAM once, so IAM will *trust tokens signed by this cluster*.
2. **An IAM role with a trust policy** that says: *"Allow this to be assumed by the OIDC provider, but only
   for ServiceAccount `<namespace>:<name>`."* (This scoping is the security bit.)
3. **A ServiceAccount annotated with the role ARN:**
   ```yaml
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: orders
     namespace: prod
     annotations:
       eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/orders-s3-role
   ```
   Any pod that uses `serviceAccountName: orders` now runs as that IAM role.

## What happens at runtime (the flow)
```text
Pod (uses ServiceAccount "orders")
  │  1. EKS injects a signed token (JWT) into the pod as a file
  ▼
AWS SDK in the pod sees AWS_ROLE_ARN + AWS_WEB_IDENTITY_TOKEN_FILE
  │  2. calls STS: AssumeRoleWithWebIdentity (presents the token)
  ▼
AWS STS
  │  3. verifies the token against the cluster's OIDC provider
  │  4. checks the role's trust policy matches "prod:orders"
  ▼
Returns TEMPORARY credentials (auto-expire, auto-refresh) → pod calls AWS
```

## Modern alternative to name
**EKS Pod Identity** (newer) does the same thing with simpler setup — no per-cluster OIDC-in-IAM wiring. But
**IRSA is still the expected interview answer**; mention Pod Identity as "the newer, simpler option."

## Summary
- **IRSA = a ServiceAccount wears an IAM role**, via the cluster's **OIDC provider**.
- Pods get **temporary, least-privilege** credentials from **STS** — no static keys, auto-rotated.
- Setup: (1) register cluster **OIDC provider** in IAM, (2) **IAM role** whose **trust policy** is scoped to
  `namespace:serviceaccount`, (3) **annotate the ServiceAccount** with the role ARN.
- Runtime: injected **JWT → `AssumeRoleWithWebIdentity` → STS temp creds**.

## Related
- [GitHub Actions OIDC → AWS federation](./26-github-actions-oidc-federation.md) — the same `AssumeRoleWithWebIdentity` pattern for **external** systems (GitHub, Databricks); trust policy + JWKS validation.
- [Pod not assuming node role / IMDS troubleshooting](./27-pod-node-role-imds-troubleshooting.md) — when a pod has **no** IRSA and falls back to the node role (IMDSv2 hop limit).
- [Authentication vs Authorization framework](./29-authentication-vs-authorization.md) — IRSA is the pod's **authN**; the assumed role's policy is the **authZ**.
