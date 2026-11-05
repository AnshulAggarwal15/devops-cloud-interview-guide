# Nielsen — DevOps / Platform Engineer Interview

> Company name as heard in the transcript sounded like **"Ningsen"** — almost certainly **Nielsen**. Correct this heading if the actual company differs.

**Round focus:** Internal Developer Platform (IDP) on the resume, Terraform (meta-arguments, providers, state, safety), and Kubernetes autoscaling — with heavy scenario/follow-up layering.

**What made it hard:** the questions were phrased in a long, indirect way with stacked follow-ups. Below, each item keeps a cleaned-up **"As asked"** version (so you can practice the phrasing) plus **"What they're really asking"**, a short answer, and a link to the full write-up.

---

## Section 1 — IDP + Terraform safety (from the resume)

### Q1.1 — Won't the platform's Terraform change break existing resources?
**As asked (paraphrased):**
> "Your IDP lets a user commit a change — say, updating the **lifecycle of an S3 bucket**. That change just **appends/updates the existing resource** in Terraform, right? So your tool won't impact or update any existing functionality of the Terraform? You gate it at the entry point with a team-lead approval — but if someone **mistakenly approves** a bad change, how do you ensure an existing resource won't get updated or destroyed?"

**What they're really asking:** your platform turns a Git commit into a `terraform apply`. What are your **guardrails** so a self-service apply only touches the intended resource and can't silently update/destroy production — especially when human approval fails?

**Short answer:**
- Terraform is declarative + stateful, so it changes **only the diff** — an S3 lifecycle edit is an **in-place update** to that one bucket. The risk is the small set of changes that **replace/destroy**.
- Guardrails, defense in depth (approval is *one* gate, not the only one):
  1. **Plan → approve the saved plan → apply that exact plan** (reviewers watch for `destroy`/`replace` lines).
  2. **Policy-as-code (OPA/Sentinel)** fails the pipeline on any destroy/replace of protected resources — catches the mistaken approval automatically.
  3. **`prevent_destroy` / `create_before_destroy`** on critical resources.
  4. **Least-privilege apply role** — if the role lacks `s3:DeleteBucket`/`rds:Delete*`, an approved-by-mistake destroy simply can't execute.
  5. **Small blast radius** — separate state per team/service/env, modular code.
- Recovery if it slips through: **revert in Git** and re-apply; **restore the versioned state** from S3 if state was corrupted; **re-import** a wrongly-deleted resource.

**Full answer:** [Terraform → IDP guardrails for safe changes](../06-terraform/15-idp-terraform-guardrails.md)
**Related:** [State file management](../06-terraform/05-statefile-management.md) · [Import blocks & state recovery](../06-terraform/103-import-blocks-and-state-recovery.md)

---

## Section 2 — Terraform deep dive

### Q2.1 — Meta-arguments; `for_each` vs `count`
**As asked (paraphrased):**
> "Give me some **meta components** in Terraform. `for_each` vs `count` — say you create five instances with each, then **delete one**. How do `for_each` and `count` behave on the deletion? And if you had to create 10 EC2s / many replicas, which would you prefer? Any other meta-arguments?"

**What they're really asking:** do you know the meta-arguments, and specifically the **index-vs-key deletion trap**?

**Short answer:**
- Meta-args: **`count`, `for_each`, `depends_on`, `provider`, `lifecycle`** (`create_before_destroy`, `prevent_destroy`, `ignore_changes`, `replace_triggered_by`).
- Delete one of five: **`count` is index-based** → removing a middle item re-indexes the tail and Terraform **destroys/recreates** the shifted ones; **`for_each` is key-based** → only the removed key is destroyed.
- Prefer **`for_each`** for many similar resources (stable under add/remove).

**Full answer:** [`count` vs `for_each` & meta-arguments](../06-terraform/12-count-vs-for_each-and-meta-arguments.md)

### Q2.2 — Same resource across different accounts / regions, and enforcing placement
**As asked (paraphrased):**
> "Create **Lambda A in AWS account A** and **Lambda B in account B** — different resources in different accounts. How do you provision that from the same Terraform? How do **modules** help? If you declare two providers, how do you **enforce** Lambda A goes to account A only and not account B? And how about the **same Lambda in a different region of the same account**?"

**What they're really asking:** provider aliasing, `assume_role` for cross-account, module provider injection, and how binding actually **enforces** placement.

**Short answer:**
- Declare **multiple `provider` blocks with `alias`** (each with its region + `assume_role` into the target account).
- **Bind** each resource with the **`provider = aws.<alias>`** meta-argument. A resource uses the **default** provider unless you point it — that explicit binding *is* the enforcement.
- **Modules:** wrap the resource, call the module once per target with **`providers = { aws = aws.account_a }`** — same code, different account/region; the module can't reach the default provider by accident.
- Same account, different region = another **aliased provider by region**.
- Extra enforcement: separate least-privilege roles per account, policy-as-code, separate state.

**Full answer:** [Multi-account & multi-region providers](../06-terraform/13-multi-account-multi-region-providers.md)

### Q2.3 — What happens on `terraform init`?
**As asked:** "What happens when you run `terraform init`?"

**Short answer:** prepares the working directory — configures the **backend** (remote state), downloads **provider plugins** and **modules**, writes the **`.terraform.lock.hcl`**. It's **safe, idempotent, and creates no infrastructure**.

**Full answer:** [What happens on `terraform init`](../06-terraform/14-terraform-init.md)

### Q2.4 — Intern breaks production; revert to a previous state? Edit state manually? How do you design for fast mitigation?
**As asked (paraphrased):**
> "Production is running. A new joiner/intern makes a **blunder in prod** and everything is down. Do you **revert to the previous Terraform state**? Do you have to **edit the state file manually** (`state` edit)? Given that problem, how would you **design your Terraform so you can quickly mitigate** an issue like this?"

**What they're really asking:** state recovery + how you architect Terraform to shrink blast radius and roll back fast.

**Short answer:**
- **First, don't hand-edit state.** Use `terraform state` commands for surgery; edit the file only as a last resort.
- **Revert path:** roll back the change in **Git** and re-apply (config is the source of truth); if state itself was corrupted, **restore the previous state version** from **S3 versioning**.
- **Design for fast mitigation:** remote backend (**S3 versioning + DynamoDB locking**), **separate state per env/service** (dev can't touch prod), **`prevent_destroy`** on critical resources, **plan review + policy-as-code** before apply, **least-privilege** apply roles, and modular code to keep the blast radius small. This is the same guardrail stack as Q1.1.

**Full answer:** [State file management](../06-terraform/05-statefile-management.md) · [Import blocks & state recovery](../06-terraform/103-import-blocks-and-state-recovery.md) · [IDP guardrails](../06-terraform/15-idp-terraform-guardrails.md)

---

## Section 3 — Kubernetes autoscaling (long scenario chain)

### Q3.1 — Deploy a stateless app and make autoscaling work seamlessly
**As asked:** "Deploy a **stateless application** — how do you deploy it, and how do you ensure **autoscaling works seamlessly**?"

**Short answer:** **Deployment** + Service, state externalized (RDS/S3/cache), set resource **`requests`** and a **readiness probe**, attach an **HPA**, and pair it with the **Cluster Autoscaler/Karpenter** so pods aren't stuck `Pending`.

**Full answer:** [Horizontal vs vertical scaling](../08-kubernetes/34-horizontal-vs-vertical-scaling.md) → then [Advanced autoscaling](../08-kubernetes/37-advanced-autoscaling-custom-metrics-keda.md)

### Q3.2 — Scale on Prometheus/application metrics (not just CPU/memory)
**As asked (paraphrased):**
> "You have a metrics system — say **Prometheus** — and you want to **scale out on the metrics your application scrapes**, not CPU/memory. How do you do it? Is there **another approach**? Show me the **pseudocode/YAML** of the first approach. And **how does the HPA identify that custom metric**?"

**What they're really asking:** custom-metrics HPA via the **Prometheus Adapter**, the alternative **KEDA**, and how the HPA discovers a custom metric (the metrics API).

**Short answer:**
- **Approach 1 — HPA + Prometheus Adapter:** the adapter registers your metric with the **`custom.metrics.k8s.io` / `external.metrics.k8s.io`** aggregated API; the HPA references the metric **by name** and the adapter translates it to a PromQL query. That registration is *how the HPA "finds" it* — it never queries Prometheus directly.
- **Approach 2 — KEDA:** event-driven `ScaledObject` (Prometheus, RabbitMQ, Kafka, SQS, cron…), can **scale to zero**; KEDA manages an HPA under the hood.
- YAML for both is in the full note.

**Full answer:** [Advanced autoscaling — custom metrics & KEDA](../08-kubernetes/37-advanced-autoscaling-custom-metrics-keda.md)

### Q3.3 — Two autoscalers on the same Deployment (HPA + event-driven) both scale at once
**As asked (paraphrased):**
> "You have a conventional **HPA** and also an **event-driven (KEDA) ScaledObject** on the **same deployment**, and both want to scale at the same time. What happens — will it scale, or will there be an issue in the cluster?"

**Short answer:** **Don't do this.** KEDA already creates/manages its own HPA, so you'd have **two controllers writing `.spec.replicas`** on one target — they **fight and thrash** (flapping up/down). **One autoscaler per workload.**

**Full answer:** [Advanced autoscaling — conflicts section](../08-kubernetes/37-advanced-autoscaling-custom-metrics-keda.md)

### Q3.4 — One event-driven autoscaler with multiple triggers — which wins?
**As asked (paraphrased):**
> "Only event-driven autoscaling, but with **multiple triggers** — Prometheus, RabbitMQ, and a Spark cluster. Trigger A wants **15** instances, trigger B wants **100**, trigger C wants to scale **down to 5**. Which trigger takes precedence — and is there any issue?"

**Short answer:** No issue — by design. KEDA computes each trigger's desired replicas and feeds one HPA, which takes the **maximum → 100**. Scaling **out wins**; a "scale to 5" trigger can't shrink you while another demands 100.

**Full answer:** [Advanced autoscaling — multiple triggers](../08-kubernetes/37-advanced-autoscaling-custom-metrics-keda.md)

### Q3.5 — Startup lag causes runaway over-provisioning
**As asked (paraphrased):**
> "The app wants to scale to **50** based on metrics, but each pod takes ~**a minute to start**. Because of the lag the metric keeps piling up and it wants to add 50 every second → over-provisioning. How do you **enforce it won't over-provision**? (And note: 50 is legitimate, `maxReplicas` is 500, yet it only scales to 50.)"

**What they're really asking:** smoothing/stabilization, not the max cap.

**Short answer:**
- **Readiness probe** so unready pods don't skew the signal.
- **HPA `behavior` / `stabilizationWindowSeconds`** + scale-up rate policies (e.g. max 10 pods/min); **KEDA** `cooldownPeriod`/`pollingInterval` for the event-driven case.
- **Not** `maxReplicas` — that's a safety ceiling. If 50 is right and max is 500, the **stabilization window** is what prevents the overshoot.

**Full answer:** [Advanced autoscaling — stabilization section](../08-kubernetes/37-advanced-autoscaling-custom-metrics-keda.md)

---

## One-line recap to rehearse
- **IDP safety:** "Approval is one gate — I add **policy-as-code, `prevent_destroy`, least-privilege apply roles, and small isolated state** so a mistaken approval can't destroy prod."
- **`count` vs `for_each`:** "`count` re-indexes and recreates on middle deletes; `for_each` is keyed and stable — I prefer `for_each`."
- **Multi-account:** "**Aliased providers** with `assume_role`, and **explicit `provider` binding / module `providers = {}`** enforces placement."
- **Prod blunder:** "Revert in Git, restore **versioned state**, never hand-edit; design with **isolated state + versioning + locking + `prevent_destroy`**."
- **Autoscaling:** "Custom metrics via **Prometheus Adapter**, events via **KEDA**; **one autoscaler per workload**; across triggers **max wins**; stop overshoot with **readiness probe + stabilization window**, not a smaller max."
