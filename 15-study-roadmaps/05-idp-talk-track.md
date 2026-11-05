# IDP Talk-Track — Your Real Stories, Spoken

Grounded in your **actual Jira-backed work**. The whole point: **steer the IDP conversation toward what you did, and own a clean honesty boundary for what the contractors built.** You have a lot to say here — this is a strength, not a weakness, once framed right.

> Rule: only put weight on what you can explain 2 layers deep. Every story below is something you actually did. Bracketed `[…]` = add your own specifics.

---

## The opening frame (say this when IDP comes up)
> "Our IDP is built on **Backstage**. The **core platform** — the frontend and backend — was built by a **contractor team**, and we adopted it while it was still in **beta**. I came in as the **SME driving adoption for my team**: I own the **Terraform-module and template/schema layer**, I onboarded **24 applications across 6 teams**, and I've spent about **1.5 years** in this codebase shipping fixes, designing platform capabilities, and leading RCAs on IDP defects. So I go deep on **how we extend and operate** the platform — modules, schemas, GitOps delivery, state safety — rather than the internals of Backstage itself."

This tells the interviewer exactly what you own. It's honest, and it's *a lot*.

## The honesty boundary (your safety net when drilled deep)
When pushed past your depth on core internals:
> "That's part of the **core platform the contractor team built** before my involvement — I'd be reasoning about it from the outside rather than from having built it. Where I can go deep is **how we extend and operate it** — let me give you a concrete example: [pivot to PUM-2521 or PE-895]."

Owning your boundary reads as **senior**. Bluffing internals reads as junior. Use this line without apology.

---

## Story 1 — Kubernetes affinity fix across IDP schemas/templates  (`PUM-2521`) ★ your best IDP story
**Say it (~90s):**
> "The IDP accepted **pod affinity settings** from teams, but during workload rendering it was **silently ignoring them** — so services weren't landing where teams intended, with no error. I owned the fix across the **schema and template layer**. I had to handle multiple affinity **operators**, **required vs preferred** affinity, **missing attributes**, **legacy string inputs** from teams already onboarded, and both **Graviton and non-Graviton** nodes — all **backward-compatible**, so I couldn't break existing consumers. I validated the rendered manifests and deployment health through **ArgoCD** in **private and stage** environments, and it shipped in **IDP release 2026.2.2**."

**If drilled:** required vs preferred affinity, why silent-ignore happened (schema→template rendering gap), how you kept legacy inputs working, how you validated via ArgoCD health across envs.
**Why it lands:** real platform-engineering — schema design, backward compatibility, multi-consumer safety, K8s scheduling.

## Story 2 — Safe Terraform state migration, prevented DB replacement  (`PE-895`) ★ your best judgment story
**Say it (~90s):**
> "Duplicate hardcoded **Terraform module labels** were blocking creation of multiple **Amazon Timestream for InfluxDB** resources. The obvious fix — renaming the module — would have made Terraform **destroy and recreate the databases**, i.e., data loss. So instead I **inventoried the blast radius**, consulted **Engineering and AWS Support**, **backed up and moved the Terraform state** so the resources kept their identity, documented the migration steps, and validated **zero replacement** and **database retention** before promoting across environments."

**If drilled:** why the rename forces replacement (identity/address change), how `plan` showed it, `terraform state mv` / moved blocks, how you proved no-replacement (plan clean, DB retained).
**Why it lands:** this is the senior instinct — you caught a destructive change before it happened. Lead with this if they probe Terraform depth.

## Story 3 — IDP stack-deletion lifecycle design  (`PUM-1501`)
**Say it (~60s):**
> "Teams could create IDP stacks but there was no safe **deletion/decommission** path. I designed the **stack-deletion lifecycle and roadmap** — turning a platform lifecycle gap into a reusable, self-service capability, with the guardrails so a deletion couldn't take out shared or protected resources."

**If drilled:** ordering of teardown, protecting stateful/shared resources, `prevent_destroy`, what a safe self-service delete needs.

## Story 4 — IDP config-drift defect + RCA  (`HCAT-1109`)
**Say it (~60s):**
> "An **adopting team was blocked** by an IDP **configuration-drift** defect. I raised and documented it, **preserved the affected service** so we didn't lose the failure state, and **coordinated the Platform team's root-cause investigation**. That preservation-for-RCA discipline is something I bring to on-call."

**If drilled:** what drift means (live state ≠ desired), why you preserved the service, how you coordinated cross-team.

## Story 5 — ArgoCD / Terraform-provider state remediation + rollback  (`PE-520`, `CM-2738`)
**Say it (~60s):**
> "We hit a **state incompatibility** between **ArgoCD** and the **Terraform provider**. I remediated it and — because it was a **state-sensitive production change** — I wrote the **deployment and rollback procedures** explicitly handling **state deletion/modification risk** before we shipped."

**If drilled:** GitOps + Terraform interaction, why state changes are risky in prod, what the rollback plan covered.

## Story 6 (delivery/GenAI, if resume comes up) — Bedrock Dockerfile review  (`PUM-1566`)
**Say it (~60s):**
> "I integrated a **Dockerfile-optimization/review service with Amazon Bedrock**, wired into Jenkins as a quality gate. I hit **IAM and pipeline blockers**, worked with engineers and AWS Support, and after evaluating latency/complexity I **chose direct model invocation over a Bedrock Agent** — a deliberate architecture trade-off. It flags security gaps and inefficient build steps pre-merge."

**If drilled:** agent-vs-direct-call trade-off (latency/config complexity), the IAM path, how it hooks into CI as a gate. (Also the OpsWorks→AMI/IDP migration `CM-1197` is a good delivery story if they want migrations.)

---

## Likely IDP drill questions → how to steer
| They ask | You steer to |
|---|---|
| "Walk me through your IDP." | The **opening frame** → then PUM-2521 or PE-895 |
| "What did *you* build vs the team?" | The **honesty boundary** — own the module/schema/template layer + adoption |
| "How does a change flow through it?" | Git commit → CI `plan` → **policy + approval gate** → Terraform apply (controlled, no raw creds) → **ArgoCD** reconciles ([../06-terraform/15-idp-terraform-guardrails.md](../06-terraform/15-idp-terraform-guardrails.md)) |
| "How do you keep a bad change from destroying prod?" | plan-review + **policy-as-code** + **`prevent_destroy`** + **least-privilege apply role** + small state (PE-895 as proof) |
| "How does the platform authenticate to AWS?" | **OIDC → STS temp creds**; workloads via **IRSA** ([../10-aws/29-...](./02-terraform-fixes-sts-oidc-and-modules.md)) |
| "Deep Backstage internals" | **Honesty boundary** → pivot to what you extend/operate |

## What NOT to claim
- Building Backstage's frontend/backend, its plugin internals, or its core scaffolder — contractor-built before you.
- "I architected the IDP." Say **"I own the module/template/schema layer and drove adoption,"** which is true and strong.

## One-liners to rehearse
- **Frame:** "Backstage-based IDP, contractor-built core; I own the **module/template/schema layer**, onboarded 24 apps across 6 teams, ~1.5 yrs in it."
- **Boundary:** "That's contractor-built core — I'd reason from outside; what I go deep on is how we **extend and operate** it."
- **Affinity fix:** "IDP silently dropped affinity; I fixed schema/template rendering backward-compatibly, shipped in 2026.2.2."
- **State safety:** "A module rename would've **recreated the InfluxDB databases** — I did a **safe state migration**, zero replacement."
- **Change safety:** "plan + policy-as-code + `prevent_destroy` + least-privilege apply role."
