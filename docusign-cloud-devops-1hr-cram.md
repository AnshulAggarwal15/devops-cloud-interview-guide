<!-- #genai: 1-hour interview cram sheet generated for Docusign Cloud/DevOps role -->

# Docusign Cloud/DevOps Interview — 1 Hour Cram Sheet

A focused, time-boxed study guide based on the job description. Read top to bottom.
Time budget is noted per section — stick to it so you finish in an hour.

**Tech stack from the JD:** AWS (networking + IAM), Terraform, GitHub Actions, EKS, Argo CD/GitOps, Python/Go. Preferred: multi-cloud, Vault, observability, AI infra (LiteLLM).

---

## Section 1 — AWS Core, Networking & IAM (15 min)

### VPC & Networking
A **VPC** is your private network in AWS. Inside it you create **subnets**:
- **Public subnet** — has a route to an **Internet Gateway (IGW)**, so resources can be reached from / reach the internet directly.
- **Private subnet** — no direct internet route. To reach the internet outbound (e.g. download packages) it goes through a **NAT Gateway** that sits in a public subnet.

**Route tables** decide where traffic goes (e.g. `0.0.0.0/0 → IGW` makes a subnet public).

**Security Groups vs NACLs** (very common question):
| | Security Group | NACL |
|---|---|---|
| Level | Instance/ENI | Subnet |
| State | **Stateful** (return traffic auto-allowed) | **Stateless** (must allow both directions) |
| Rules | Allow only | Allow + Deny |

### IAM
- **User** = a person/long-lived identity. **Role** = temporary identity that something *assumes* (no permanent credentials). **Policy** = JSON document granting permissions.
- A **role** has a **trust policy** (who can assume it) + **permission policy** (what it can do).
- Prefer roles over hard-coded keys everywhere.

### IRSA (IAM Roles for Service Accounts) — memorize this
This is how a **pod in EKS gets AWS permissions securely**, without putting access keys in the pod.
- EKS has an **OIDC provider**. You map a Kubernetes **ServiceAccount** to an **IAM role**.
- Pods using that ServiceAccount automatically get temporary AWS credentials.

### Core services to name comfortably
EC2 (compute), S3 (object storage), ALB/NLB (load balancers), Route 53 (DNS), ECR (container registry), CloudWatch (metrics/logs), RDS (managed DB).

---

## Section 2 — Terraform (15 min — they want "advanced")

### What it is
Terraform is **Infrastructure as Code** — you declare desired infra in `.tf` files, run `plan` (preview) then `apply` (execute). It's **declarative**: you describe the end state, Terraform figures out the changes.

### Modules
A **module** is a reusable package of Terraform config (inputs → resources → outputs).
- **Inputs (variables):** parameters you pass in.
- **Outputs:** values the module exposes (e.g. a VPC ID) for other modules to consume.
- **Why:** reuse the same pattern (e.g. "a standard VPC") across many environments. Version modules so changes are controlled.

### State management (key topic)
Terraform tracks what it created in a **state file**.
- Store state **remotely**, not locally — typically **S3 backend + DynamoDB for locking**.
- **Locking** (DynamoDB) prevents two people running `apply` at the same time and corrupting state.
- **Drift** = when real infra differs from state (someone changed it manually); `terraform plan` detects it.
- Useful: `terraform state list`, `terraform state mv`, `terraform import`.

### Things to be ready to mention
- `count` vs `for_each` — `for_each` is preferred for creating multiple resources from a map/set (stable addressing).
- `lifecycle` block — `prevent_destroy`, `create_before_destroy`.
- Workspaces — multiple states from one config (one way to separate envs).

**Likely Q — "How do you manage state across a team?"**
→ Remote backend in S3, state locking via DynamoDB, separate state per environment, never commit state to Git (it can hold secrets).

---

## Section 3 — Kubernetes / EKS + Argo CD + GitOps (12 min)

### EKS basics
- **EKS** = AWS-managed Kubernetes. AWS runs the **control plane**; you manage **worker nodes** (managed node groups or Fargate).
- Scaling nodes: **Cluster Autoscaler** or **Karpenter** (newer, faster, more flexible).
- Pods get AWS access via **IRSA** (see Section 1).

### Core objects (1-liners)
- **Pod** — smallest unit, wraps containers.
- **Deployment** — manages ReplicaSets, keeps N pods running, handles rolling updates.
- **Service** — stable IP/DNS + load balancing for a set of pods.
- **Ingress** — HTTP/HTTPS routing into the cluster (needs an ingress controller).
- **ConfigMap / Secret** — inject config / sensitive data.

### GitOps
**Principle:** Git is the **single source of truth** for both app and infra. You change the cluster by committing to Git; a controller continuously **reconciles** the cluster to match Git.

### Argo CD
- A GitOps controller for Kubernetes. It watches a Git repo of manifests and **syncs** them to the cluster.
- **Self-heal:** if someone changes the cluster manually, Argo CD reverts it back to match Git.
- **Drift detection:** shows "OutOfSync" when cluster ≠ Git.

**Likely Q — "What is GitOps and how does Argo CD implement it?"**
→ Desired state lives in Git; Argo CD watches Git and reconciles the cluster to match it, with auto-sync, self-heal, and drift detection.

---

## Section 3.5 — Helm & Docker (8 min — don't skip, these are common gaps)

### Helm (pairs directly with Argo CD)
**Helm** is the package manager for Kubernetes. A **chart** is a packaged set of templated manifests.
- **Templates + `values.yaml`** — you write manifests with placeholders; `values.yaml` supplies the actual values. This lets you deploy the same chart to dev/staging/prod with different values.
- **Release** — an installed instance of a chart in the cluster.
- **Why it matters here:** Argo CD commonly deploys **Helm charts** (or **Kustomize** overlays). GitOps + Helm = templated, versioned, environment-specific deployments from Git.
- **Kustomize** (alternative): overlays/patches plain YAML per environment — no templating language. Know it exists.

### Docker fundamentals
- **Image vs container:** image = read-only blueprint; container = running instance.
- **Dockerfile basics:** `FROM` (base image), `RUN` (build steps), `COPY`, `CMD`/`ENTRYPOINT` (what runs).
- **Multi-stage builds (important):** use one stage to build (with compilers/deps) and a slim final stage that only copies the artifact → **smaller, more secure images**.
- **Best practices:** small base images (alpine/distroless), pin versions, leverage layer caching (put rarely-changed steps first), don't bake secrets into images, run as non-root.

---

## Section 3.6 — Deployment Strategies (4 min — classic question)

| Strategy | How it works | Trade-off |
|---|---|---|
| **Rolling update** (K8s default) | Replace pods gradually, a few at a time | No downtime; brief mix of old+new versions |
| **Blue-Green** | Run new (green) alongside old (blue), switch traffic at once | Instant rollback; needs 2x resources |
| **Canary** | Send a small % of traffic to the new version, then ramp up | Safe, gradual; needs traffic control (e.g. service mesh/ingress) |

**Likely Q — "How do you roll back a bad deploy?"**
→ Rolling: `kubectl rollout undo`. Blue-green: switch traffic back to blue. GitOps/Argo CD: revert the Git commit and let Argo CD sync to the previous state.

---

## Section 4 — CI/CD with GitHub Actions (10 min)

### Structure
A **workflow** (YAML in `.github/workflows/`) is made of:
- **Triggers** (`on:`) — e.g. push, pull_request, schedule.
- **Jobs** — run on **runners** (VMs); jobs run in parallel by default.
- **Steps** — commands or reusable **actions** within a job.

### Key concepts
- **Secrets:** stored in repo/org settings, referenced as `${{ secrets.NAME }}`.
- **OIDC to AWS (important):** GitHub Actions can authenticate to AWS using **OIDC federation** — short-lived credentials, **no long-lived access keys stored**. This is the modern, secure approach.
- **Reusable workflows** and **matrix builds** (test across multiple versions in parallel).

**Likely Q — "How do you securely authenticate GitHub Actions to AWS?"**
→ Use OIDC: configure an IAM role with a trust policy for GitHub's OIDC provider; the workflow assumes the role and gets temporary credentials. No static keys.

---

## Section 5 — Python / Go for Automation (6 min)
You likely won't get a deep coding round, but be ready to talk through:
- **boto3** (AWS SDK for Python) — scripting AWS tasks (e.g. list/stop instances, manage S3).
- **Idempotency** — scripts should be safe to re-run (check before create).
- **Error handling** — handle API failures/retries gracefully.
- Go is common for infra tooling because it compiles to a single binary and is fast (kubectl, Terraform, k8s itself are written in Go).

---

## Section 6 — Preferred / Bonus (skim, 2 min)
- **Multi-cloud** — same IaC/GitOps principles across AWS/Azure/GCP; abstract provider-specific bits.
- **HashiCorp Vault** — central secrets management; supports **dynamic secrets** (short-lived, generated on demand) vs static stored secrets.
- **Observability — the three pillars:** **metrics** (Prometheus), **logs** (ELK/Loki), **traces** (Jaeger/OpenTelemetry); visualize with **Grafana**.
- **LiteLLM** — a proxy/gateway giving a **unified API across many LLM providers** (OpenAI, Anthropic, etc.), with routing, rate limiting, and cost tracking. Relevant for provisioning AI/ML infra.

---

## Section 7 — Kubernetes Troubleshooting Quick Recall (3 min)

| Symptom | First thing to check | Common cause |
|---|---|---|
| **CrashLoopBackOff** | Logs (`kubectl logs --previous`) | App crashing on start, bad config, missing env/secret, OOM |
| **ImagePullBackOff** | Pod events (`kubectl describe pod`) | Wrong image name/tag, no registry/ECR auth |
| **Pending** | `kubectl describe pod` (events) | No node capacity, unschedulable (taints/affinity), no PVC |
| **Service unreachable** | Selector matches pod labels? Endpoints exist? | Label mismatch, wrong port, readiness failing |

**Method:** `describe` (events) → `logs` → check config/secrets → check resources/networking. Don't stop at the first cause.

---

## Section 8 — 30-Second Behavioral Opener (2 min)
Have a crisp answer for **"Tell me about your experience"**:
> "I have X years in DevOps/Platform engineering. I work day-to-day with **AWS** (VPC, IAM, EKS), provision infra with **Terraform** using reusable modules and remote state, run CI/CD on **GitHub Actions**, and deploy to **EKS** using **Argo CD with a GitOps workflow**. I automate tooling in **Python**."

Tailor the specifics to your real experience — but lead with the exact stack in the JD.

---

## The 5 Must-Know Answers (review last, memorize)
1. **IRSA** — pods get AWS permissions securely via OIDC + ServiceAccount→IAM role mapping. No keys in pods.
2. **Terraform remote state + locking** — S3 for state, DynamoDB for lock; prevents corruption, enables teamwork.
3. **GitOps via Argo CD** — Git is source of truth; controller reconciles cluster, self-heals, detects drift.
4. **GitHub Actions → AWS via OIDC** — short-lived creds, no static keys.
5. **Security Group vs NACL** — SG = instance-level, stateful; NACL = subnet-level, stateless.

---

## Likely Scenario Question (rehearse out loud)
**"Walk me through deploying an app to EKS using Terraform, GitHub Actions, and Argo CD."**

1. **Terraform** provisions infrastructure — VPC, subnets, EKS cluster, IAM roles (with remote state in S3 + DynamoDB lock).
2. **GitHub Actions** builds the app image, pushes it to **ECR**, and updates the Kubernetes manifests in the Git repo (authenticating to AWS via OIDC).
3. **Argo CD** detects the manifest change in Git and **syncs** it to the EKS cluster, with self-heal keeping the live state matching Git.

---

*Good luck — read the 5 Must-Know answers one final time right before you walk in.*
