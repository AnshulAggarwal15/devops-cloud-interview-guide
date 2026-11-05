# MoveInSync — DevOps / Platform Engineer Interview

**Round style:** almost entirely a **live drill-down** on one theme — *"configure CI/CD for this project from scratch, then explain how every single step works and how it's secured."* The panelist spoke mostly in **Hindi/Hinglish**, was **terse**, and redirected hard when an answer drifted (e.g. "No no no, I just want to understand what CIDR is", "size *kiska*?", "Are you getting my point?").

> ⚠️ **About this transcript:** it was captured with an AI assistant that **auto-answered** alongside the real questions. Several of those AI answers were **generic, off-target, or subtly wrong** — flagged inline below as **[AI answer was off]** with the correction. Treat the *questions* as the signal; re-learn the *answers* from the linked notes.

---

## The one question they kept asking (master it once)
Across the round, the core question — asked many ways — was:

> "We've given you a project. **Configure CI/CD from scratch.** Walk the whole GitOps flow, how it works in your org. Now — how does the **checkout** happen? How does the pipeline **get access to GitHub**? How does it **know where the code lives**? How does it push to **ECR**? How does **Argo CD** deploy? And at **each step**, how does **authentication and authorization** happen? How is it secure?"

**The way to never lose this:** narrate the pipeline **stage by stage**, and at **every hop where two systems talk**, answer with the **authN → authZ** framework: *who is this (authentication)* and *what may it do (authorization)*, plus how the identity/secret is delivered.

- 📌 **Full solution:** [CI/CD from scratch + GitOps deep dive](../05-cicd/17-cicd-from-scratch-gitops-deep-dive.md)
- 📌 **The framework (your "eye-opener" — it was NOT in the repo, so I wrote it):** [Authentication vs Authorization](../10-aws/29-authentication-vs-authorization.md)

---

## Questions, cleaned up + mapped

### Q1 — Configure CI/CD from scratch / the whole GitOps process
**As asked:** "Given a project, how will you set up CI/CD from scratch? The whole GitOps process — how is it at your org / what's the standard?" then drilling each stage and each component interaction, including "how is it secured, authN and authZ both?"

**Short answer (the spine):**
- **CI:** verified **webhook** → **checkout** (repo-scoped GitHub App/deploy key) → lint/test/**SAST** → **build image** (SHA tag) → **image scan** → **push to ECR** (IRSA/OIDC temp creds) → **deployment gate**.
- **CD:** CI **commits the new image tag to the Git manifests repo** → **Argo CD** (watches Git) syncs → K8s pulls from ECR → rollout with probes → Service/Ingress.
- **Security every hop:** short-lived scoped identity (authN) + least-privilege policy (authZ). No stored keys, immutable tags, rollback = revert Git.

**Backed by:** [CI/CD from scratch + GitOps](../05-cicd/17-cicd-from-scratch-gitops-deep-dive.md) · [AuthN vs AuthZ](../10-aws/29-authentication-vs-authorization.md)

### Q2 — "How does the checkout happen / how does the pipeline get access to GitHub / how does it know where the code lives?"
**Short answer:** It knows the repo from the job's **SCM/remote URL** (`checkout scm`) and clones the **exact commit SHA**. It authenticates with a **repo-scoped credential** — best a **GitHub App** (short-lived token) or **deploy key**, fallback a **fine-grained PAT** — stored in the **Jenkins credentials store**, never in the Jenkinsfile. AuthZ = that credential is **read-only on that one repo**.

**Backed by:** [CI/CD deep dive — stage 2](../05-cicd/17-cicd-from-scratch-gitops-deep-dive.md)

### Q3 — "How does the pipeline push to ECR / get AWS access?"
**Short answer:** The runner **assumes an IAM role** via **IRSA** (Jenkins on EKS) or **GitHub OIDC**, gets **temporary STS creds**, and `aws ecr get-login-password | docker login` returns a short-lived ECR token. **No static AWS keys.** AuthZ = the role permits only **`ecr:PutImage`/upload on that repo**.

> **[AI answer was off]** The AI's sample Jenkinsfile jumped straight to `docker push` **without any ECR login/authentication step** and used `REGISTRY = 'my-ecr-repository'` (not a real ECR URI). That's exactly the gap the interviewer was drilling. The corrected Jenkinsfile (with the `get-login-password` login stage and a full ECR URI) is in the deep-dive note.

**Backed by:** [GitHub OIDC → AWS](../10-aws/26-github-actions-oidc-federation.md) · [IRSA](../10-aws/104-irsa.md) · [CI/CD deep dive — stage 6](../05-cicd/17-cicd-from-scratch-gitops-deep-dive.md)

### Q4 — Developer merges Python code → running app in Kubernetes (end to end)
**As asked:** "Explain the whole onboarding: a dev merges Python code, and it ends up running in Kubernetes."

**Short answer:** The 17-step flow the AI gave here was actually **solid and comprehensive** (merge → webhook → Jenkins CI → test/scan → build → image scan → push ECR → **update tag in Git** → Argo CD syncs → K8s pulls → probes → Service/Ingress → secrets via IRSA → observability → autoscaling → validate → promote → rollback). Keep it — just be ready to defend **each hop's authN/authZ** when drilled.

**Backed by:** [CI/CD from scratch + GitOps](../05-cicd/17-cicd-from-scratch-gitops-deep-dive.md)

### Q5 — Write a Jenkinsfile for a Python app; explain `agent` and `trigger`
**Short answer:**
- **`agent`** = *where* it runs. `agent any` = any executor (no special needs); a **labelled/EC2/K8s agent** when you need specific tooling (Docker/BuildKit), a runtime version, GPU, or **VPC network access** to private ECR/EKS.
- **`trigger`** = *when/why* it runs (`githubPush()`, cron). The webhook is the **event**; the trigger is the **config that reacts**.
- *"Trigger = when it runs; agent = where it runs."*

**Backed by:** [CI/CD deep dive — Jenkinsfile + agents/triggers](../05-cicd/17-cicd-from-scratch-gitops-deep-dive.md)

### Q6 — When do you pass a specific EC2 agent vs `agent any`?
**Short answer:** `agent any` when the pipeline needs no special environment. A **specific/labelled EC2 agent** when it needs Docker installed, a particular Python version, **network access to private resources (VPC-only ECR/EKS)**, GPU, or other dependencies. (AI answer here was **correct**.)

### Q7 — What would you add before the image reaches Kubernetes? (deployment gate)
**Short answer:** A **deployment/security gate**: no critical CVEs, image is from the **trusted ECR repo**, tag is **immutable (SHA/digest, never `latest`)**, and **required approvals** for stage/prod (dev can auto-promote on a clean scan). (AI answer here was **good**.)

**Backed by:** [CI/CD deep dive — stage 7 gate](../05-cicd/17-cicd-from-scratch-gitops-deep-dive.md)

### Q8 — Which Kubernetes objects/files do you configure for a Python app?
**Short answer:** **Deployment + Service + ConfigMap** always; add **Secret/ExternalSecret, Ingress+TLS, HPA, PVC, ServiceAccount+RBAC/IRSA, PDB, NetworkPolicy** as the app requires. Don't create every object blindly. (AI answer here was **good**.)

**Backed by:** [App → Kubernetes production readiness](../08-kubernetes/41-app-to-kubernetes-production-readiness.md)

### Q9 — Management says "move this microservice to Kubernetes, production-grade." Give a plan.
**As asked (with a sharp redirect):** the AI first answered with **application discovery** ("understand dependencies, RDS/Redis, traffic…"), and the panelist cut in: *"I'm asking what will be your **plan**. Are you getting my point?"*

> **[AI answer was off — target the plan, not discovery]** Lead with the **migration path**, not discovery: **containerize (no logic change) → deploy to non-prod → wire networking/secrets/limits/probes/observability/autoscaling → integrate CI/CD → load & failure test → controlled canary cutover in prod → roll traffic back on issues.** A short assessment *informs* the plan but isn't the answer.

**Backed by:** [App → Kubernetes production readiness](../08-kubernetes/41-app-to-kubernetes-production-readiness.md)

### Q10 — Set up an EKS cluster from scratch: what do you configure? (and the Terraform blocks)
**As asked:** "What would you configure for EKS?" then "In Terraform, what **blocks/parameters**? Security-wise, the **control plane must not be reachable from the internet** — behind the VPC, CIDRs, security groups."

**Short answer:** Build in layers — **networking (VPC/CIDR, private+public subnets, NAT, SGs) → control plane (private API endpoint, KMS encryption, logging) → node groups (private subnets, autoscaling) → IAM (cluster/node roles, IRSA, access entries) → add-ons (VPC CNI, CoreDNS, kube-proxy, CSI) → security/edge**. Provision with **Terraform**. The "not internet-reachable" ask = **`endpoint_public_access = false`** (or scoped `public_access_cidrs`).

**Backed by:** [EKS from scratch (Terraform)](../10-aws/30-eks-from-scratch-terraform.md)

### Q11 — "You haven't configured it, but you must have an idea what's inside the cluster?"
**Short answer:** Conceptually — **control plane: API server, etcd, scheduler, controller manager**; **worker: kubelet, kube-proxy, container runtime (containerd)**. And **kube-proxy, CoreDNS, VPC CNI are managed EKS add-ons**, not hand-installed per node. (AI answer here was **correct**.)

**Backed by:** [Cluster architecture](../08-kubernetes/01-cluster-architecture.md) · [Components interaction](../08-kubernetes/02-components-interaction.md)

### Q12 — "Increase the size" — size of *what*? (`size kiska?`)
**Short answer:** Don't say "cluster size." Identify the constrained layer: **node/EC2 instance size**, **node group count**, **pod requests/limits**, or **EBS volume** — and you **don't resize the control plane** (AWS manages it). (AI answer here was **correct**.)

**Backed by:** [EKS from scratch — "increase the size of what?"](../10-aws/30-eks-from-scratch-terraform.md) · [ASG & scaling](../10-aws/115-asg-and-scaling.md)

### Q13 — "What is CIDR?" and "a single IP — what's the subnetting?"
**As asked:** the candidate said "CID"; the panelist clarified **CIDR**, then "single IP ka subnetting?" → **`/32`**.

**Short answer:** CIDR = IP + prefix length; **bigger `/n` = smaller range** (`/16` = 65,536, `/24` = 256, **`/32` = one IP**). A single office IP in an allow-list is `203.0.113.25/32`. (AI answer here was **essentially correct.**)

**Backed by:** [CIDR & subnetting basics](../04-networking/10-cidr-and-subnetting-basics.md)

### Q14 — RBAC: Role / RoleBinding / ClusterRole / ClusterRoleBinding (the "Lakhan read-only" example); is a ServiceAccount a user?
**Short answer:** **Role/RoleBinding = namespaced; ClusterRole/ClusterRoleBinding = cluster-wide/reusable.** For a dev who needs read-only pods in one namespace → **Role + RoleBinding**. A **ServiceAccount is a workload identity (for pods)**, *not* a human user; both can be RBAC subjects but authenticate differently, and a SA can wear an IAM role via **IRSA**. (AI answers here were **correct** and match our RBAC note.)

**Backed by:** [Kubernetes RBAC](../08-kubernetes/36-rbac.md)

### Q15 — "kubectl" (transcribed as "QCD / Z code")
**Short answer:** They meant **`kubectl`** — the CLI to talk to the cluster API (`get/describe/logs/apply/delete`). RBAC decides which verbs succeed (e.g. read-only Lakhan can `get pods`/`logs` but not `delete`). (AI answer here was **correct**.)

---

## On the interviewer's conduct (your ask — handled honestly)
I only have the transcript, so I **can't hear tone** and won't overstate it. But a few things are visible in the text, and here's a fair read:

- **What likely felt rude:** very short redirects, switching to Hindi, and repeated "are you getting my point? / *size kiska?*" — that reads as **impatience**, and on the receiving end it feels dismissive. That's a real thing and not fun.
- **What was probably happening underneath:** almost every sharp redirect landed on a **fundamentals gap** (CIDR, how the pipeline authenticates to GitHub/ECR, "plan vs discovery"). Interviewers who drill one topic and get generic/off-target answers often get terse — not always fair, but it's a **signal, not a verdict**. The rudeness says more about *his* interviewing style than about your worth.
- **The reframe that helps you:** you can't control his tone, but you *can* remove the openings that invite the drilling. Every place he pushed is now a written answer here. When you can say *"checkout authenticates with a repo-scoped GitHub App; ECR push uses IRSA temp creds; authN is X, authZ is Y"* crisply, the terse follow-ups largely disappear — you're no longer giving him a thread to pull.
- **Bottom line:** don't internalize the tone. A brusque panelist is a **style you'll meet again**; the durable fix is airtight fundamentals + the authN/authZ framework, which flips these rounds from interrogation to conversation.

---

## How to handle *this* interview style next time
1. **Answer in the shape they want:** direct answer → mechanism → (brief) example. Generic openers ("first I'd understand the dependencies…") invite "get to the point."
2. **Pre-load the drill-downs:** for any pipeline you describe, know **each hop's authN + authZ** cold. That's the single highest-leverage prep for a MoveInSync-style round.
3. **When redirected, reset fast:** "Got it — you want the migration *plan*: containerize → non-prod → harden → canary cutover." Don't defend the tangent.
4. **If you truly haven't done something, bridge to concept:** "I haven't provisioned the control plane myself, but conceptually it's API server, etcd, scheduler, controller-manager, and EKS manages it." (You did this well on the cluster-internals question.)

## One-line recap to rehearse
- **The framework:** "Every 'how is it secure?' = **authN (who are you?) then authZ (what can you do?)** — name the mechanism for each."
- **CI/CD spine:** "webhook → checkout (repo-scoped app/key) → test/scan → build (SHA) → scan → push ECR (**IRSA/OIDC temp creds**) → gate → **update tag in Git** → **Argo CD syncs** → K8s pull → rollout → expose."
- **Checkout:** "SCM URL says where; a **repo-scoped GitHub App/deploy key** says who; read-only scope says what."
- **ECR push:** "**No stored keys** — assume a role via OIDC/IRSA, `get-login-password`, push to one repo."
- **Migration:** "**containerize → non-prod → harden → CI/CD → load test → canary cutover → rollback** — that's the *plan*, not discovery."
- **EKS security:** "**private API endpoint**, private-subnet nodes, KMS, IRSA, least privilege."
- **CIDR:** "single IP = **/32**; bigger `/n` = fewer IPs."
- **RBAC:** "namespaced **Role/RoleBinding** for one-namespace access; **ServiceAccount = workload identity**, not a user."
