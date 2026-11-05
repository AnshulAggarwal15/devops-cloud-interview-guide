<!-- #genai: AWS interview note - EKS pod not getting AWS creds via node instance role; IMDS / IMDSv2 hop limit troubleshooting -->

## Question
An EKS pod has **no IRSA** and is expected to fall back to the **node's IAM role** (instance profile), but it's **not working**. The pod is scheduled and the node is fine. Where do you start looking?

### Short Explanation
Without IRSA, the AWS SDK in a pod tries to get credentials from the **node's instance metadata service (IMDS)** at `169.254.169.254`. On EKS this commonly fails for two reasons: the **IMDSv2 hop limit** is too low for a pod (the packet needs an extra hop out of the pod network), or **IMDS access is deliberately blocked/restricted**. So you debug the **credential chain and the IMDS path**, not the app.

## Answer
Start by confirming *how* the SDK is trying to get credentials, then test the IMDS path from inside the pod:
1. Check the pod isn't accidentally on **IRSA** (a ServiceAccount annotated with `eks.amazonaws.com/role-arn` injects a token that takes precedence).
2. From inside the pod, hit **IMDSv2** (`PUT` for a token, then `GET`) at `169.254.169.254` — if that fails, it's the metadata path.
3. The usual culprit: **IMDSv2 hop limit = 1**. A pod's request needs **2 hops**, so set the instance metadata **`httpPutResponseHopLimit` to 2** (or 3). Also check IMDS isn't disabled/blocked.
4. Verify the **node instance role** actually has the needed permissions and a correct trust policy.

## Detailed Explanation

### The credential chain (know the order)
The AWS SDK resolves credentials in order: env vars → **web identity token (IRSA)** → shared config → container creds → **EC2 instance metadata (IMDS = node role)**. So:
- If **IRSA is partially configured** (SA annotated, token file present) the SDK uses that and *never* reaches the node role — and if that role/trust is wrong, it fails "mysteriously." Check `AWS_WEB_IDENTITY_TOKEN_FILE` / `AWS_ROLE_ARN` env in the pod.
- If you truly want the **node role**, none of the earlier providers should be set.

### Test IMDS from inside the pod (IMDSv2)
```bash
# IMDSv2 requires a session token first (PUT), then use it (GET)
TOKEN=$(curl -sX PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 60")
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/iam/security-credentials/
```
- Times out / no route → **network/hop-limit/block** issue.
- Returns nothing for `iam/security-credentials/` → the node has **no instance role** attached.
- Works but app still fails → the role exists but **lacks permissions** (check the policy).

### The #1 cause: IMDSv2 hop limit
- IMDSv2 sets an IP **TTL/hop limit** on responses. Default **`httpPutResponseHopLimit = 1`** works for a process on the host, but a **pod sits behind an extra network hop**, so the response's TTL hits 0 and is dropped.
- Fix: set the launch template / instance metadata options to **hop limit 2** (some setups use 3):
```text
MetadataOptions:
  HttpTokens: required          # IMDSv2
  HttpPutResponseHopLimit: 2    # <- lets pods reach IMDS
  HttpEndpoint: enabled
```

### Other things to check (in order)
| Check | Why it breaks creds |
|---|---|
| SA annotation `eks.amazonaws.com/role-arn` | Triggers IRSA; SDK ignores node role (and may fail on that role) |
| `HttpPutResponseHopLimit` | Default 1 blocks pods → set 2 |
| `HttpEndpoint: disabled` / IMDS blocked | Many hardened clusters block IMDS from pods on purpose |
| Network policy / `iptables` to `169.254.169.254` | CNI or security tooling can drop metadata traffic |
| Node instance role permissions | Role attached but missing the needed action |
| Node role **trust policy** | Must trust `ec2.amazonaws.com` |
| `AWS_EC2_METADATA_DISABLED=true` | Env var turns IMDS off in the SDK |

### The security angle (say this)
Blocking pod access to IMDS is a **best practice** (stops a compromised pod from stealing the node's broad role) — that's exactly why **IRSA / EKS Pod Identity** exist. So "pod relying on the node role" is usually a design smell; the real fix is often **give the pod its own IRSA role**, not open up IMDS.

## Summary
- No IRSA → SDK falls back to **IMDS (node role)**. Debug the **credential chain** then the **IMDS path**.
- Most common fix: **IMDSv2 `httpPutResponseHopLimit = 2`** (pods need an extra hop); also check IMDS isn't disabled/blocked and the node role's permissions/trust.
- Watch for a stray **SA role-arn annotation** silently forcing IRSA. Best practice is to **give the pod its own IRSA role** rather than depend on the node role.

## Related
- [IRSA](./104-irsa.md) · [GitHub/OIDC federation](./26-github-actions-oidc-federation.md)
- [IAM users, roles & policies](./103-iam-users-roles-policies.md)
