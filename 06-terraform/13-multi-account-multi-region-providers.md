## Multi-Account & Multi-Region Provisioning in Terraform

### Question  
How do you provision the same/similar resources across **different AWS accounts** (Lambda A in account A, Lambda B in account B) or **different regions** in the **same** account — and how do you **enforce** that each resource lands in the right place?

### Short explanation of the question  
This tests **provider aliasing**. One Terraform config can talk to many accounts/regions by declaring **multiple `provider` blocks with `alias`**, and you **explicitly bind** each resource/module to a provider via the **`provider` meta-argument**. The enforcement part is the catch: a resource goes to a provider **only if you point it there** — get the alias wrong and it lands in the wrong account.

---

### Answer  
- Declare **multiple provider configurations** using **`alias`** (one per account/region), each authenticating to its target (region + credentials/`assume_role`).
- Attach each resource or module to the intended provider with the **`provider = aws.<alias>`** meta-argument.
- **Enforcement** = explicit binding: a resource uses the **default** provider unless you set `provider = aws.<alias>`. For modules, pass providers explicitly via the **`providers = { ... }`** map so the module can't accidentally use the default.
- Cross-account access is done by having each provider **`assume_role`** into the target account.

---

### Detailed explanation of the answer for readers’ understanding

### 🔀 Multiple accounts via aliased providers + `assume_role`
```hcl
provider "aws" {
  alias  = "account_a"
  region = "us-east-1"
  assume_role { role_arn = "arn:aws:iam::1111:role/tf-deployer" }
}

provider "aws" {
  alias  = "account_b"
  region = "us-east-1"
  assume_role { role_arn = "arn:aws:iam::2222:role/tf-deployer" }
}

resource "aws_lambda_function" "lambda_a" {
  provider      = aws.account_a          # <- lands in account A only
  function_name = "lambda-a"
  # ...
}

resource "aws_lambda_function" "lambda_b" {
  provider      = aws.account_b          # <- lands in account B only
  function_name = "lambda-b"
  # ...
}
```
Each provider assumes a role in its own account; the **`provider`** meta-argument is what pins the resource.

### 🌍 Same account, different region
Same idea, aliased by **region** (no `assume_role` needed if it's one account):
```hcl
provider "aws" { region = "us-east-1" }              # default

provider "aws" {
  alias  = "eu"
  region = "eu-west-1"
}

resource "aws_lambda_function" "lambda_eu" {
  provider      = aws.eu                 # same account, eu-west-1
  function_name = "lambda-eu"
}
```

### 📦 How modules help (reuse the same code per target)
Wrap the resource in a module, then **call it once per account/region**, passing the right provider:
```hcl
module "lambda_account_a" {
  source    = "./modules/lambda"
  providers = { aws = aws.account_a }    # inject provider into the module
  name      = "lambda-a"
}

module "lambda_account_b" {
  source    = "./modules/lambda"
  providers = { aws = aws.account_b }
  name      = "lambda-b"
}
```
Same module code, different account/region — DRY and consistent. Inside the module you declare a `configuration_aliases` (or a required `aws` provider) so callers must pass one.

### 🔒 How you *enforce* the right target
- **Explicit `provider` binding**: without `provider = aws.<alias>`, a resource uses the **default** provider. So the enforcement is "always set the alias, and only give the default provider to what belongs there."
- **`providers = {}` on modules**: the module can only use what you pass in — it can't reach the default provider by accident.
- **Separate roles per account** (`assume_role`) with least privilege — even a mis-pointed resource can't create in an account whose role you didn't grant.
- Guardrails: **policy-as-code (OPA/Sentinel)** and code review to catch a wrong alias before apply; separate state per account to shrink blast radius.

### ⚠️ Things they probe
- The default provider vs aliased providers — resources silently use the default if you forget the alias.
- `assume_role` is the standard **cross-account** mechanism (the deployer role must be trusted by the target account).
- Provider config **can't be dynamic per-`for_each`-key** easily — for many accounts, generate calls or use a stack tool (Terragrunt) per account.

---

### 🧪 Real-World Example

> “We deployed the same Lambda + IAM setup into three AWS accounts from one repo. We defined **aliased providers** that each `assume_role` into the target account, wrapped the resources in a **module**, and called the module three times with `providers = { aws = aws.<account> }`. Nothing could land in the wrong account because each call only ever saw its injected provider.”

---

### Key takeaway

> “Use **aliased `provider` blocks** (one per account/region, cross-account via `assume_role`) and **bind each resource/module explicitly** with the `provider` meta-argument or a module `providers = {}` map. A resource only goes where you point it — the default provider is the trap. Modules let you reuse the same code per target.”

### Related
- [`count` vs `for_each` & meta-arguments](./12-count-vs-for_each-and-meta-arguments.md) — the `provider` meta-argument
- [Terraform modules](./02-terraform-modules.md)
- [AWS STS / assume-role](../10-aws/18-aws-sts.md)
