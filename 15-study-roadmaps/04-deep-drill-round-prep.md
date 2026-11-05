# Deep-Drill Round Prep — IDP · Dockerfile · Secret Rotation · EKS Reliability · DB · Coding · Nerves

Read Section 0 first (nerves — do it now), then the topic you're weakest on. Each drill topic gives you **the underlying model → the drill-down follow-ups → the crisp answer**, so you can go two layers deep instead of freezing.

---

## 0) NERVES — do this in the next 5 minutes (and during the call)
You're shivering because part of you is scared of being "caught not knowing." **Kill that fear at the root: decide *right now* that you're allowed to say "I don't know, here's how I'd approach it."** That one decision drops the pressure more than any fact.

**Right now (2 min):**
- **Box breathing ×4:** inhale 4s → hold 4s → exhale **6s** (long exhale calms the nervous system) → hold 2s. Repeat 4 times. The long exhale is the switch.
- **Warm your hands** (cold hands = adrenaline). Feet flat on floor. Sit up. **Drink water.**
- **Say out loud once:** *"I've prepared. I'll pause before I answer. I'll be honest. It's a conversation, not an exam."*

**During the call:**
- **The pause is your anchor.** Every question: breathe once, say *"let me think for a second,"* then answer. Nerves make you rush and ramble — the pause is the cure and it reads as *senior*.
- **Slow your speech down ~20%.** Nervous = fast + high-pitched. Slow + lower = calm + credible. Water sip = a legal reset button.
- **Start strong:** have your 60-sec "tell me about yourself" rehearsed cold — nailing the first 2 minutes settles the whole call.
- **If you blank:** "Can I take a moment?" or "Could you rephrase that?" — totally normal, buys calm.
- **Reframe the interviewer:** he has a seat to fill; he *wants* you to be good. He's not hunting you.

Keep 3 anchor phrases on a sticky note in view: **"Pause." "Direct answer → mechanism → example." "Honest beats bluff."**

---

## 1) IDP (Internal Developer Platform) — your must-know, drilled hard before
> ⚠️ Map this to **YOUR** actual IDP. This is the canonical model + the exact questions you were asked at Nielsen/Amagi. Fill in your real tools where they differ.

### The one-line definition
"An IDP is a **self-service layer** that gives developers a **paved/golden path** to ship and run services — they request or change infrastructure and deployments through the platform, and it handles provisioning, deployment, and guardrails so they don't touch raw Terraform/Kubernetes/AWS."

### The canonical architecture (be able to draw this)
```text
Developer
  │  self-service (portal / Backstage / a Git template / CLI)
  ▼
Service catalog + golden-path TEMPLATE  (scaffolds repo + IaC + pipeline)
  │  developer commits a change (e.g., "add an S3 bucket", bump config)
  ▼
Git (source of truth)  ──►  CI (Jenkins/Actions): validate + plan
  │                              │  policy checks + approval gate
  ▼                              ▼
IaC engine (Terraform)  ──apply──►  AWS (VPC/EKS/RDS/S3/IAM…)
  │
  ▼
GitOps (Argo CD) deploys the app to EKS  ──►  observability + guardrails
```

### How it works underneath (the "how" they drill)
- **Self-service front door:** a portal (Backstage) / template repo / CLI. Developer picks a golden path and fills inputs.
- **Templating/scaffolding:** generates a repo with a standard pipeline + IaC module calls, so every service is consistent (this is the "paved road").
- **Change flow = GitOps + IaC:** a developer change is a **Git commit** → CI runs **`terraform plan`** (and app CI) → **policy + approval gate** → **`terraform apply`** through a controlled service (the dev never has raw cloud creds) → **Argo CD** reconciles app deployments.
- **Guardrails (the security story — lead with this at a finance firm):** plan-review + **approve the saved plan**, **policy-as-code (OPA/Sentinel)** blocking destroy/replace of protected resources, **`prevent_destroy`** on stateful resources, **least-privilege apply role** (so an approved-by-mistake destroy still can't execute), **separate state per team/env** (small blast radius). *(Deep note: [../06-terraform/15-idp-terraform-guardrails.md](../06-terraform/15-idp-terraform-guardrails.md))*
- **Identity:** the platform's pipeline authenticates to AWS via **OIDC → STS temp creds** (no stored keys); workloads get AWS via **IRSA**. *(See STS/OIDC in [02-...](./02-terraform-fixes-sts-oidc-and-modules.md))*

### The actual questions you got asked (with crisp answers)
- **"A dev commits a change to an S3 lifecycle — does your tool touch existing resources?"** → "Terraform is declarative + stateful, so it changes **only the diff** — a lifecycle edit is an **in-place update** to that one bucket. The risk is the small set of changes that **replace/destroy**, which is why we gate on the plan + policy + `prevent_destroy`."
- **"If a bad change is mistakenly approved, how do you stop it destroying prod?"** → "Approval is one gate, not the only one: **policy-as-code fails the pipeline** on destroy/replace of protected resources, **`prevent_destroy`** blocks it in Terraform, and the **least-privilege apply role** can't delete stateful resources even if asked. Recovery = **revert the Git commit** / restore versioned state."
- **"How does the platform get access to the customer/AWS account (external system)?"** → "A role alone isn't assumable by an external system — we use **OIDC federation + a scoped trust policy**, then `AssumeRoleWithWebIdentity` for temporary creds."

### Drill-down follow-ups to be ready for
- "Why an IDP vs letting devs use Terraform directly?" → consistency, guardrails, reduced cognitive load, security (no raw creds), auditability.
- "How do you onboard a new service?" → golden-path template scaffolds repo + pipeline + IaC.
- "How do you handle secrets in the platform?" → external secret store + IRSA, never in Git (see §3).
- "What breaks / what's hard about running an IDP?" → drift, template versioning, keeping guardrails from blocking legit work, multi-team state.

---

## 2) Dockerfile — build, optimize, secure (and AI/ML angle)
### The mental model
Image = **immutable, layered blueprint**; container = a **running instance**. Each Dockerfile instruction is a **cached layer**; a change busts that layer **and all after it**.

### Optimize (the drill: "make this image smaller/faster")
1. **Layer order for cache (biggest win):** copy dependency manifest → install → *then* copy source, so code changes don't reinstall deps.
   ```dockerfile
   COPY requirements.txt .
   RUN pip install --no-cache-dir -r requirements.txt   # cached unless deps change
   COPY . .
   ```
2. **Multi-stage build:** build in a heavy stage, copy only the artifact into a slim final image (build tools never ship).
3. **Small/trusted base:** `-slim` / **distroless** → smaller + fewer CVEs. Pin the tag/digest, not `latest`.
4. **`.dockerignore`:** keep `.git`, tests, secrets out of the build context.
5. **Fewer layers:** combine related `RUN`s and clean up in the same layer (`apt-get ... && rm -rf /var/lib/apt/lists/*`).
6. **Security:** **non-root `USER`**, no secrets baked in (build args/mounts or runtime injection), **scan with Trivy**, read-only rootfs where possible.

### `CMD` vs `ENTRYPOINT` (common quick drill)
- **ENTRYPOINT** = the fixed executable; **CMD** = default args (overridable). Use ENTRYPOINT for the binary, CMD for default flags. *(Note: [../07-docker/10-cmd-vs-entrypoint.md](../07-docker/10-cmd-vs-entrypoint.md))*

### AI/ML image specifics (if the "AI" part is literal)
- Base images are **huge** (CUDA/torch) → multi-stage hard, so pin a slim runtime, strip build/compilers, use `--no-cache`.
- **Don't bake model weights into the image** if large/changing — **download at runtime** or mount from S3/EFS (keeps images small + rebuilds fast).
- GPU needs the **NVIDIA container runtime**; pin CUDA/driver compatibility.
- Scan Python/ML deps (supply-chain risk is real).

**Full note:** [../07-docker/13-image-vs-container-and-optimizing-dockerfiles.md](../07-docker/13-image-vs-container-and-optimizing-dockerfiles.md)

---

## 3) Secret rotation (great deep topic — finance loves it)
### Why rotate
Limit the blast radius of a leaked credential: short-lived secrets are useless after rotation, and rotation forces you to *prove* your apps can handle credential changes.

### How rotation actually works — AWS Secrets Manager (the mechanism they drill)
Secrets Manager rotates via a **Lambda** and **staged versions** (`AWSCURRENT`, `AWSPENDING`, `AWSPREVIOUS`), in **4 steps**:
1. **createSecret** — generate a new secret value, store it as **`AWSPENDING`**.
2. **setSecret** — set the new value in the target service (e.g., create the new DB password).
3. **testSecret** — verify the new value works (connect with it).
4. **finishSecret** — promote `AWSPENDING` → **`AWSCURRENT`** (old becomes `AWSPREVIOUS`).
- For **RDS**, Secrets Manager has managed rotation (single-user or **two-user** strategy — two-user alternates between two DB users for **zero-downtime** rotation).

### How apps pick up a rotated secret (the follow-up that trips people)
- Apps must **re-read** the secret — options: **fetch on each use / cache with short TTL**, **restart/rollout** the pods (`kubectl rollout restart`), or a **sidecar/agent** that refreshes it.
- **Zero-downtime rotation** = overlap: the old and new secret are **both valid for a grace window** (two-user strategy / dual keys) so in-flight connections don't break.

### Vault angle (rotation via short TTL)
- Vault issues **dynamic secrets** (e.g., a unique DB user per app with a **lease/TTL**); "rotation" is really **expiry + reissue** — nothing static to rotate. This is the strongest model.

### Kubernetes angle
- Secrets synced from Secrets Manager/Vault via the **External Secrets Operator** (or CSI driver); on rotation it updates the K8s Secret, and you **restart/reload** consumers to pick it up.

### Drill follow-ups
- "How do you rotate without downtime?" → two-user/dual-secret overlap + short-TTL caching.
- "Where do you store the secret?" → Secrets Manager/Vault, encrypted (KMS); never in Git/image; K8s Secrets are only **base64** → back with encryption-at-rest + external store.
- "How does the pod get the secret without keys?" → **IRSA** → reads Secrets Manager.

**Notes:** [../10-aws/22-secrets-manager-vs-parameter-store.md](../10-aws/22-secrets-manager-vs-parameter-store.md) · [../10-aws/119-vault-secrets-management.md](../10-aws/119-vault-secrets-management.md)

---

## 4) Production reliability on EKS (finance = reliability + blast radius)
### The mental model
Reliability = **no single point of failure + graceful failure + self-healing + controlled change**, at every layer: nodes, pods, traffic, data, deploys.

### The layers (be able to list these fast)
- **Multi-AZ:** node groups across **≥2–3 AZs**; control plane is multi-AZ managed by AWS.
- **Redundancy:** multiple **replicas** spread across AZs (**topology spread constraints** / pod anti-affinity).
- **Disruption protection:** **PodDisruptionBudget** (keep min replicas during drains/upgrades); **graceful shutdown** (`preStop` hook + `terminationGracePeriodSeconds`, drain connections).
- **Health:** **readiness** (don't send traffic early) + **liveness** (restart hangs) + **startup** probes.
- **Right-sizing:** **requests/limits** (requests drive scheduling; limits → throttle/OOM) → **QoS**.
- **Autoscaling:** **HPA** (pods) + **Cluster Autoscaler/Karpenter** (nodes); **KEDA** for event-driven.
- **Safe deploys:** **rolling** or **canary/blue-green**; **rollback** = revert Git (GitOps).
- **Self-healing:** ReplicaSet recreates dead pods; ASG/managed node group replaces dead nodes; node **NotReady** → pods evicted + rescheduled.
- **Data/DR:** stateful data in **RDS Multi-AZ** (not in-cluster if avoidable); backups/PITR; PVCs are AZ-bound.
- **Security posture:** **IRSA** (no static keys), **NetworkPolicies** (default-deny), **Pod Security**, image scanning, private API endpoint. *(Notes: [../08-kubernetes/40-cluster-hardening.md](../08-kubernetes/40-cluster-hardening.md), [../08-kubernetes/41-app-to-kubernetes-production-readiness.md](../08-kubernetes/41-app-to-kubernetes-production-readiness.md))*
- **Observability + SLOs:** metrics/logs/traces, **SLIs/SLOs + error-budget** alerting (burn rate, not raw CPU). *(Notes: [../09-observability/13-sli-slo-sla-error-budgets.md](../09-observability/13-sli-slo-sla-error-budgets.md), [14-incident-management-and-p1.md](../09-observability/14-incident-management-and-p1.md))*

### The "10x traffic" / "how do you make it reliable" answer
"Spread the **stateless** tier across AZs behind a load balancer, autoscale it (HPA + Cluster Autoscaler), protect it with **PDBs + probes + topology spread**, and shield the **stateful** DB with **caching + read replicas + connection pooling** — the DB is almost always the bottleneck. Deploy via canary with automatic rollback, and alert on **SLO burn rate**."

### Drill follow-ups
- "What happens during a node upgrade?" → cordon + drain (honors PDB) → pods reschedule → new node. *(Note: [../08-kubernetes/47-cordon-drain-vs-taint.md](../08-kubernetes/47-cordon-drain-vs-taint.md))*
- "How do you avoid dropping requests on deploy?" → readiness probe + `preStop` + graceful termination + rolling/canary.
- "Single replica on a dead node?" → downtime until rescheduled → run **multi-replica + PDB + spread**.

**Full note:** [../10-aws/107-high-availability-and-autoscaling.md](../10-aws/107-high-availability-and-autoscaling.md)

---

## 5) DB — the ~15-minute deep version (more than the one-liners)
### SQL you must be fluent in
- **Joins:** INNER (matches both), LEFT (all left + matches), RIGHT, FULL. Be ready to *write* one.
- **GROUP BY + HAVING** (HAVING filters *after* aggregation; WHERE filters *before*).
- **Indexes:** B-tree index speeds `WHERE`/`JOIN`/`ORDER BY`; **composite index** column order matters (leftmost prefix); **covering index** serves a query from the index alone. Cost: slower writes + storage. Too many indexes hurt.
- **`EXPLAIN` / query plan:** how you find why a query is slow — look for **full table scans**, missing index usage, bad join order, high row estimates.
- **N+1 problem:** app fires a query per row instead of one join/batch — classic ORM latency bug.
- **Window functions** (basic): `ROW_NUMBER() OVER (PARTITION BY … ORDER BY …)` for "top N per group", running totals.

### Transactions & concurrency
- **ACID:** Atomic (all-or-nothing), Consistent (valid state), Isolated (concurrent txns don't corrupt), Durable (committed = persisted).
- **Isolation levels:** Read Uncommitted → Read Committed (default many DBs) → Repeatable Read → Serializable. Higher = safer, less concurrency. Know the anomalies: dirty read, non-repeatable read, phantom.
- **Locking & deadlocks:** two txns wait on each other's locks → DB kills one; avoid by consistent lock ordering + short txns.

### Design & scale
- **Normalization** (reduce redundancy, 3NF) vs **denormalization** (fewer joins, faster reads — for read-heavy/reporting).
- **Replication vs Multi-AZ (say this crisply):** **Multi-AZ = synchronous standby for failover/HA (not readable)**; **read replicas = async copies for read scaling (readable, can be cross-region)**. Don't confuse them.
- **Partitioning** (split one table by range/hash) vs **sharding** (split across servers) — for very large data.
- **Connection pooling / RDS Proxy:** DB connections are expensive; pool them; RDS Proxy helps serverless/Lambda avoid exhausting connections.
- **Backups:** automated snapshots + transaction logs → **PITR**; test restores.
- **SQL vs NoSQL:** relational/ACID/joins vs flexible schema/horizontal scale/simple access patterns (DynamoDB) — pick by access pattern.
- **CAP** (for distributed DBs): under a partition, choose consistency or availability.

### The "debug a slow query" playbook (a likely question)
1. Find the slow query (slow query log / RDS Performance Insights).
2. **`EXPLAIN`** it → full scan? missing index? bad plan?
3. Add/fix an **index** (or rewrite the query, avoid `SELECT *`, avoid N+1).
4. Check **locks/contention**, **connection limits**, and whether it's a **read** you can offload to a **replica/cache**.
5. Re-measure.

### If your deep SQL is rusty — the honest line
"I'm stronger on the **operational** side — RDS Multi-AZ vs replicas, backups, connection pooling, sizing — and comfortable with core SQL (joins, indexes, `EXPLAIN`); I haven't done heavy query tuning, but here's how I'd approach a slow query…" (then the playbook).

---

## 6) Coding — the 15-min live/screen-share task
### What to expect
A small **Python or Bash** problem (or SQL). Not LeetCode-hard usually — they watch **how you think**, communicate, and handle edge cases.

### The method (this matters more than the answer)
1. **Clarify** the problem + inputs/outputs + edge cases before coding (buys calm + shows seniority).
2. **State your approach out loud** in one sentence before typing.
3. **Write simple, correct code** — clarity over cleverness. Name things well.
4. **Test as you go** — print, run small inputs. Don't write 30 lines silently.
5. **Talk the whole time** — narrate what you're doing. Silence + shivering is the failure mode; narration keeps you calm and shows reasoning.
6. **Mention edge cases** at the end (empty input, duplicates, large input).

### Likely problems + approach (review these)
- **Count word/line frequency in a log file** → read lines, split, dict/`collections.Counter`, sort by count.
- **Find the most frequent element / first non-repeating** → hash map counts.
- **Parse a log for ERROR lines in the last hour** → read, filter by substring + timestamp.
- **Two Sum / find pair summing to target** → hash set of seen values (O(n)).
- **Reverse a string / check palindrome / FizzBuzz** → warm-up basics; don't overthink.
- **boto3 automation** → "list EC2 instances and stop untagged ones" → paginate, filter, act; **idempotent** + error handling.
- **Bash:** delete/list files older than N days (`find /path -mtime +30 -delete`), count occurrences (`grep -c`), top IPs in a log (`awk '{print $1}' | sort | uniq -c | sort -nr | head`).

### Screen-share tips
- Increase font size; keep it readable. Have a scratch area for approach.
- If you blank: write the **brute-force** first, say "I'll optimize after it works." A working simple solution beats a broken clever one.
- If stuck: "Let me trace through a small example" — narrate; the interviewer often nudges.

**Practice source:** [../12-python/05-python-programming-interview-questions.md](../12-python/05-python-programming-interview-questions.md) (do 2–3 from memory before the call).

---

## The 7 things to rehearse OUT LOUD before you join
1. 60-sec **"tell me about yourself."**
2. 90-sec **IDP walkthrough** (architecture-first) + the guardrails one-liner.
3. **Multi-AZ vs read replica** (one sentence).
4. **Dockerfile optimize** (layer cache + multi-stage + non-root).
5. **Secret rotation** (4-step Secrets Manager + "apps re-read / two-user zero-downtime").
6. **EKS reliability** (multi-AZ + replicas + PDB + probes + autoscale + rollback).
7. One **Bash one-liner** + the **coding method** ("clarify → approach → simple code → test → edges").

And the meta: **pause, direct-answer-first, honest-beats-bluff.** You've got real material here — trust it, slow down, and let the prep carry you.
