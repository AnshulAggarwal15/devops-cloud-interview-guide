# Terraform Gap-Fixes + AWS STS/OIDC (cross-domain) + Module Structure

Companion to your Terraform drill (baseline ~6.5/10 — solid mid-level). Three parts:
1. **Brief answers to what you missed** (~5-min read)
2. **AWS STS + OIDC** — the one federation model that covers **Terraform, Kubernetes (IRSA), and CI/CD**
3. **Module structure** + a concrete **deployment child module**

---

## Part 1 — The questions you missed (brief)

**What's actually IN state / what it's used for** (you drifted into locking)
State does four functional things: (1) **maps** each config address → real provider ID (`aws_instance.web` → `i-0abc`); (2) stores resource **attributes/metadata** so `plan` can diff without always re-reading the cloud; (3) tracks **dependencies** (create/update/destroy order); (4) stores **outputs**. Locking/backup are *management* concerns, not what state is *for*.

**`for` expression** (not the same as `for_each`)
An **expression** that transforms/filters a collection into a new list or map. It builds *values*; it does not create resources.
```hcl
[for s in var.names : upper(s)]                 # list → list
{ for k, v in var.map : k => v.id }             # map → map
[for s in var.names : s if s != ""]             # with a filter
```

**`dynamic` block**
Generates **repeated nested blocks** inside a resource from a collection — use it when a resource has a repeatable sub-block (SG rules, `setting`, `ingress`).
```hcl
dynamic "ingress" {
  for_each = var.rules
  content {
    from_port = ingress.value.port
    to_port   = ingress.value.port
    protocol  = "tcp"
  }
}
```

**Provisioners** (and why they're a last resort)
`local-exec` runs a command **on the machine running Terraform**; `remote-exec` runs **on the created resource** (SSH/WinRM). Last resort because they're **imperative**, **not tracked in state**, and fail non-idempotently — they break the declarative model. Prefer `user_data`/cloud-init, baked images (Packer), or config management.

**`replace_triggered_by`**
A `lifecycle` arg that **forces replacement** of this resource when a referenced resource/attribute changes (e.g., recreate an instance when a config hash changes).

**Passing a provider into a module** (you fumbled this)
Use the **`providers` map** in the module block; the module declares it needs one via `configuration_aliases`. Needed for **multi-account/region** modules.
```hcl
# module declares it (versions.tf inside the module)
terraform { required_providers { aws = { source = "hashicorp/aws", configuration_aliases = [aws.target] } } }

# root passes it
module "app_account_b" {
  source    = "./modules/app"
  providers = { aws.target = aws.account_b }
}
```

**`terraform_remote_state`** (cross-stack outputs)
A data source that reads **another stack's outputs** from its backend:
```hcl
data "terraform_remote_state" "net" {
  backend = "s3"
  config  = { bucket = "tf-state", key = "network/prod.tfstate", region = "us-east-1" }
}
# use: data.terraform_remote_state.net.outputs.vpc_id
```
(Looser alternative: publish IDs to **SSM Parameter Store** and read them — less coupling between stacks.)

**Secrets handling** (you only said "store state remotely")
- **Don't hardcode.** Mark inputs `sensitive = true` — but know this only hides them from CLI output; **state still stores them in plaintext.**
- So: **encrypt state at rest** (S3 SSE-KMS), restrict backend access (IAM), and pull secrets **at runtime** from **Secrets Manager / SSM / Vault** (data source or the app itself) rather than baking them into config/state.
- Best: **generate** short-lived secrets (Vault) so nothing durable lives in state.

**Testing / validation tooling + the gate** (you were unsure)
- `terraform fmt` (style) → `terraform validate` (syntax/config) → **`tflint`** (best-practice/provider lint) → **`checkov` / `tfsec` / `terrascan`** (security/misconfig) → **`terraform plan`** (review a *saved* plan) → **`terratest` / `terraform test`** (spin real infra, assert, destroy).
- **Gate:** run all of these in **CI**, **fail** on lint/scan/plan errors, and require **plan approval** (+ policy-as-code OPA/Sentinel) before prod apply.

**Provider version guarantee** (you missed the lock file)
`required_providers` sets the **allowed range**; the **`.terraform.lock.hcl`** file records the **exact resolved versions + checksums**. **Commit the lock file** so every dev and CI run installs identical providers. `terraform init` respects it; `terraform init -upgrade` updates it.

**Refactor without destroy/recreate** (you used the old way)
Prefer the declarative **`moved {}` block** over `terraform state mv`:
```hcl
moved { from = aws_instance.old  to = aws_instance.new }
```

**Workspaces vs separate dirs — the *why*** (you had the preference, not the reason)
Workspaces share **one backend + one codebase**, just separate state per workspace → cheap, but **weak isolation** (easy to apply to the wrong one; same code/backend = bigger blast radius). Good for ephemeral/dev. **Separate directories (or state keys) per env** = different backends/vars, **can't fat-finger prod**, stronger isolation → prefer for prod.

---

## Part 2 — AWS STS + OIDC (the model that covers TF, K8s & CI/CD)

**This one concept is your highest-leverage fix** — it was a hole in *both* your K8s (IRSA) and Terraform (keyless CI) answers, and it's drilled everywhere.

### The universal idea
A workload proves *who it is* with a **signed OIDC token (JWT)**; **AWS STS** exchanges that token for **temporary credentials** via **`sts:AssumeRoleWithWebIdentity`**, gated by an **IAM role trust policy** that checks the token's claims. **No long-lived keys anywhere.**

### The flow (memorize this shape — it's identical every time)
```text
1. An OIDC Identity Provider signs a JWT for the workload   (claims: iss, sub, aud)
2. You register that IdP once in IAM (OIDC identity provider)
3. An IAM role's TRUST POLICY allows sts:AssumeRoleWithWebIdentity
   only for that issuer + a specific sub + aud
4. Workload calls AssumeRoleWithWebIdentity with its JWT
     → STS verifies the signature against the IdP's JWKS
     → STS checks iss / sub / aud against the trust policy
     → returns TEMPORARY credentials (auto-expire)
5. Workload uses temp creds; no static keys exist
```
- **authN = the JWT** ("who are you") · **authZ = the role's permission policy** ("what can you do") · **trust policy = who may assume the role**.
- `401`-style failure = bad/untrusted token; `403`-style = authenticated but the policy denies.

### The three flavors — same model, only `iss`/`sub` differ
| | Identity Provider (`iss`) | Subject (`sub`) scopes to… | Extra bits |
|---|---|---|---|
| **CI/CD (GitHub → AWS)** | `token.actions.githubusercontent.com` | `repo:org/repo:ref:refs/heads/main` (repo/branch/env) | workflow needs `permissions: id-token: write` + `aws-actions/configure-aws-credentials`; `aud=sts.amazonaws.com` |
| **Kubernetes (EKS IRSA)** | the **EKS cluster's** OIDC provider URL | `system:serviceaccount:<namespace>:<sa>` | annotate the SA with the role ARN; EKS projects the SA token; SDK auto-assumes. Newer/simpler: **EKS Pod Identity** |
| **Terraform (cross-account)** | (enters via GitHub OIDC above) | — | provider blocks then **`assume_role`** into each target account (plain `sts:AssumeRole`, account-to-account trust + optional `external_id`) |

### How they chain in a real Terraform pipeline
```text
GitHub Actions workflow
  → GitHub OIDC JWT → AssumeRoleWithWebIdentity → temp creds in the HUB account   (keyless entry)
    → Terraform provider `assume_role` into ACCOUNT A  (AssumeRole, cross-account)
    → Terraform provider `assume_role` into ACCOUNT B
```
So **keyless entry = `AssumeRoleWithWebIdentity` (OIDC)**; **hopping between accounts = `AssumeRole` (account trust)**. That distinction is a great thing to say out loud.

### The one sentence to have cold (works for all three)
> "A signed **OIDC JWT** identifies the workload; **STS validates it against the registered IdP and the role's trust policy** (`iss`/`sub`/`aud`) and hands back **temporary credentials** — no long-lived keys. Only the **issuer and subject** change: **GitHub repo/branch** for CI, **`namespace:serviceaccount`** for EKS IRSA."

### Tiny examples
**GitHub → AWS trust policy (CI/CD):**
```json
{ "Effect":"Allow",
  "Principal":{"Federated":"arn:aws:iam::111:oidc-provider/token.actions.githubusercontent.com"},
  "Action":"sts:AssumeRoleWithWebIdentity",
  "Condition":{
    "StringEquals":{"token.actions.githubusercontent.com:aud":"sts.amazonaws.com"},
    "StringLike":{"token.actions.githubusercontent.com:sub":"repo:my-org/my-repo:ref:refs/heads/main"}}}
```
**IRSA trust policy (EKS):** same shape, but `Federated` = the cluster OIDC ARN and the condition matches `...:sub` = `system:serviceaccount:prod:payments`.
**Terraform cross-account provider:**
```hcl
provider "aws" { alias = "account_b" region = "us-east-1"
  assume_role { role_arn = "arn:aws:iam::222:role/tf-deployer" } }
```

*(Deeper write-ups already exist: [GitHub OIDC](../10-aws/26-github-actions-oidc-federation.md) · [IRSA](../10-aws/104-irsa.md) · [authN vs authZ](../10-aws/29-authentication-vs-authorization.md).)*

---

## Part 3 — Module structure (and a deployment child module)

### Standard module layout (every module looks like this)
```text
modules/<name>/
├── main.tf         # the resources
├── variables.tf    # inputs (typed, with descriptions/defaults/validation)
├── outputs.tf      # what callers can consume
├── versions.tf     # terraform + required_providers (+ configuration_aliases)
└── README.md       # inputs/outputs/usage
```

### Repo layout (root/"live" calls the modules)
```text
live/                      # root configs per environment (state lives here)
├── dev/   ( main.tf + backend.tf + dev.tfvars )
├── stage/
└── prod/
modules/                   # reusable building blocks (no backend, no state)
├── vpc/
├── eks/
└── app-deployment/        # ← the child module below
```
Key rule: **root/live modules own the backend + provider config + state; child modules own resources only** and receive everything via variables/providers.

### A deployment child module — `modules/app-deployment/`
A reusable module that deploys one app/service. This example deploys a **Kubernetes Deployment + Service** (swap the resources for ECS/Lambda if that's your target — the *shape* is the same).

`versions.tf`
```hcl
terraform {
  required_version = ">= 1.5"
  required_providers {
    kubernetes = {
      source                = "hashicorp/kubernetes"
      configuration_aliases = [kubernetes.target]   # caller passes which cluster
    }
  }
}
```
`variables.tf`
```hcl
variable "name"       { type = string }
variable "namespace"  { type = string  default = "default" }
variable "image"      { type = string }                 # immutable tag / digest
variable "replicas"   { type = number  default = 2 }
variable "port"       { type = number  default = 8080 }
variable "cpu"        { type = string  default = "250m" }
variable "memory"     { type = string  default = "256Mi" }
variable "env"        { type = map(string) default = {} }
```
`main.tf` (abridged — shows probes, requests/limits, dynamic env)
```hcl
resource "kubernetes_deployment" "this" {
  provider = kubernetes.target
  metadata { name = var.name  namespace = var.namespace }
  spec {
    replicas = var.replicas
    selector { match_labels = { app = var.name } }
    template {
      metadata { labels = { app = var.name } }
      spec {
        container {
          name  = var.name
          image = var.image
          port { container_port = var.port }
          resources {
            requests = { cpu = var.cpu, memory = var.memory }
            limits   = { cpu = var.cpu, memory = var.memory }
          }
          readiness_probe { http_get { path = "/healthz" port = var.port } }
          dynamic "env" {
            for_each = var.env
            content { name = env.key  value = env.value }
          }
        }
      }
    }
  }
}

resource "kubernetes_service" "this" {
  provider = kubernetes.target
  metadata { name = var.name  namespace = var.namespace }
  spec {
    selector = { app = var.name }
    port { port = 80  target_port = var.port }
    type = "ClusterIP"
  }
}
```
`outputs.tf`
```hcl
output "service_name" { value = kubernetes_service.this.metadata[0].name }
output "namespace"    { value = var.namespace }
```

### How the root calls it (note the `providers` map — Part 1's gap)
```hcl
module "orders_api" {
  source    = "../../modules/app-deployment"
  providers = { kubernetes.target = kubernetes.prod_cluster }   # inject the target cluster
  name      = "orders-api"
  namespace = "prod"
  image     = "123.dkr.ecr.us-east-1.amazonaws.com/orders:${var.git_sha}"   # immutable tag
  replicas  = 4
  env       = { LOG_LEVEL = "info" }
}
```

### What makes it a *good* module (say this in an interview)
- **Single responsibility** (one thing well), **typed variables with validation + sensible defaults**, **clear outputs**, **no hardcoded providers/backends** (injected), **versioned** (Git tag / registry), and a **README**. It should be reusable across envs by changing only inputs.

---

## Your 3 highest-leverage Terraform fixes (in order)
1. **STS + OIDC** (Part 2) — fixes Terraform *and* K8s *and* CI/CD in one shot.
2. **Module provider-passing + `terraform_remote_state`** (Part 1 + Part 3) — you use modules but couldn't wire providers/cross-stack.
3. **Process maturity** — secrets handling, `tflint`/`checkov`/`terratest` gate, and the **lock file**.
