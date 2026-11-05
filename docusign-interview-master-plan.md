

# Docusign Interview — Master Plan

Single source of truth for the two **pure technical** rounds:

- **Round 1 (in ~20h):** Coding + System Design
- **Round 2 (~20h after R1):** Cloud + IaC + GitOps

This plan **schedules** your time and **links** to material you already wrote. It does not repeat it. Keep these two cram sheets open — they are your primary revision docs:

- [docusign-r2-coding-systemdesign-cram.md](./docusign-r2-coding-systemdesign-cram.md) — use for **Round 1** (it is coding + system design despite the "r2" filename).
- [docusign-cloud-devops-1hr-cram.md](./docusign-cloud-devops-1hr-cram.md) — use for **Round 2**.
- [docusign-debug-order-playbook.md](./docusign-debug-order-playbook.md) — cross-cutting: the **order to debug** AWS / Kubernetes / CI/CD / performance scenarios (one framework + per-scenario checklists). Skim before any troubleshooting question.
- [docusign-request-lifecycle-handbook.md](./docusign-request-lifecycle-handbook.md) — the **full request lifecycle** (client → Route 53 → ALB → Ingress → Service → Pod → DB → back), naming every component, plus CI/CD, Terraform, and observability context. The one diagram to recall for "walk me through what happens when…" senior questions.

> How to read the schedule: times are **offsets from now** (`T-20h` = now, `T-0` = round starts). Each window includes one sleep and a final morning revision. Shift the clock labels to your actual start time.

> Progress tracking: tick `- [ ]` → `- [x]` as you finish each item. `[x]` = done, `[~]` = partially done / in progress. Check off a warm-up or problem only after you've **written and run it**, not just read it.

---

## 1. Mindset & scoring (read first, re-read before each round)

They are hiring a **Developer Platform Engineer**. They are answering one question: *"Can this person think through problems, communicate clearly, and operate real platforms?"* — not *"Do they memorize trivia?"*

**The 5-step answer pattern — use it on every question (coding, design, and scenario):**

1. **Clarify** — restate the problem, ask 1-2 questions (scale? input size? edge cases? constraints?).
2. **State assumptions** out loud.
3. **Approach / architecture** — brute force first for coding; high-level diagram for design.
4. **Trade-offs** — "it depends on whether we optimize for X or Y" is a senior answer.
5. **Operate / debug** — how you'd monitor it and recover from failure.

**Two power-moves (from your R2 cram):**

- **Think out loud.** Say the approach before you code. They score reasoning, not clean syntax.
- **Name the gap, then pivot to concept.** *"I haven't built this hands-on, but here's how I'd approach it and why."* The recruiter explicitly said this is welcome.

**Honesty scripts (use without shame):**

- *"I haven't implemented this hands-on, but conceptually here's how I'd approach it…"*
- *"Let me reason from fundamentals…"*
- *"There's a trade-off here — it depends on whether we optimize for X or Y."*

**Golden rule for these 40 hours:** do **not** learn new tech. Become *fluent* in what you already know. Polish, don't expand. Spend most of your time **explaining aloud**, not passively reading.

---

## 2. Where the marks are (weight your effort here)

Estimated probability a topic comes up (from the interview analysis):

- **Your own projects — ~100%** (highest ROI; they will make you talk about them)
- **Kubernetes — ~95%**
- **Terraform / IaC — ~95%**
- **AWS — ~90%**
- **Python scripting — ~80%**
- **GitHub Actions / CI-CD — ~80%**
- **GitOps / Argo CD — ~80%**
- **Coding (easy–medium) — ~70%**
- **System Design — ~70%**
- Go — ~20% (don't spend time here)

Takeaway: **80% of your prep value is system design + explaining your own projects + core platform scenarios.** Coding is real but easy-to-medium; don't over-invest in algorithms.

---

## 3. WINDOW 1 — 20 hours to Round 1 (Coding + System Design)

Target: ~9.5h focused work + break + 7-8h sleep + 1.5h morning revision. Adjust clock to your start.

### Block 1 — Python fluency & idioms · 1.5h · `T-20h → T-18.5h`

**Status: fluency reading DONE (~2h via Python blogs); warm-up coding drills PENDING → do these next.**

Fluency / reading:

- [x] Read fluency material: R2 cram **Part A (A1–A3)** + [12-python](./12-python) — covered via [InterviewBit Python cheat sheet](https://www.interviewbit.com/python-cheat-sheet/), [chanduthedev beginner](https://chanduthedev.medium.com/python-interview-questions-for-beginners-cd05ddda40af), [intermediate](https://chanduthedev.medium.com/python-interview-questions-intermediate-level-c7ce66342e2d), [advanced](https://chanduthedev.medium.com/python-interview-questions-advanced-2dabff728e54).
- [x] Idioms recognized: `list / dict / set / tuple` (mutable? ordered? duplicates?), comprehensions, `enumerate`, `zip`, `sorted()` vs `.sort()`, `lambda / map / filter`, `try/except/finally`, `Counter`, `defaultdict`, context managers (`with`), generators (`yield`), deep vs shallow copy.
- [ ] **Concept Q&A drill:** [12-python/04-python-conceptual-interview-questions.md](./12-python/04-python-conceptual-interview-questions.md) (mutable/immutable, generators, decorators, GIL, deep vs shallow copy, OOP, MRO — real chanduthedev-series questions + additions).
- [ ] **Coded problems drill:** [12-python/05-python-programming-interview-questions.md](./12-python/05-python-programming-interview-questions.md) (stack/two-pointer/hash-map/generator problems with worked solutions; real Altimetrik/Foraysoft/TrackMind questions + Two Sum/anagrams/log parsing).

Warm-up — write from scratch, no lookups (THIS is what's still pending):

- [ ] Count frequencies (`Counter` / dict)
- [ ] Reverse a string (`s[::-1]`)
- [ ] Merge two dicts
- [ ] Group anagrams
- [ ] Read JSON and extract a field

Exit check:

- [ ] Can write any warm-up above in <2 min without syntax lookups.

### Block 2 — Coding practice · 2.5h · `T-18.5h → T-16h`

Solve in this order (LeetCode Easy). For **each**: brute force out loud → optimize (can a dict/set kill the nested loop?) → code → test 1 normal + 1 edge case → state time/space.

Core problems (the first five carry the "hash map kills the nested loop" instinct — prioritize):

- [ ] 1. Two Sum
- [ ] 2. Valid Anagram
- [ ] 3. Contains Duplicate
- [ ] 4. First Unique Character
- [ ] 5. Valid Parentheses
- [ ] 6. Best Time to Buy and Sell Stock
- [ ] 7. Merge Sorted Array
- [ ] 8. Reverse Linked List
- [ ] 9. Middle of the Linked List
- [ ] 10. Binary Search

DevOps-flavored variants (very likely — practice at least one):

- [ ] Count HTTP status codes in a log file (stream with `with open(...) as f:` so memory stays flat)
- [ ] Find duplicate / matching lines in a log
- [ ] Read a CSV/JSON and extract fields
- [ ] Call an API and parse the response

Support + exit check:

- [ ] Review reference solutions/talking tracks: R2 cram **Part A (A4–A5)**.
- [ ] **Exit check:** Two Sum / Valid Parentheses / First Unique from memory + correct Big-O for each.

### Break · 20m · `T-16h → T-15.7h`

Get away from the screen.

### Block 3 — System design framework + building blocks · 1h · `T-15.7h → T-14.7h`

- [ ] Read R2 cram **Part B (B1–B3)**.
- [ ] Building blocks fluent: Load Balancer, stateless app tier, SQL vs NoSQL, cache (Redis), CDN, message queue (SQS/Kafka), object storage (S3).
- [ ] Framework memorized: clarify → scale numbers → high-level diagram → bottlenecks → trade-offs.
- [ ] Instincts memorized: stateless app tier + LB = horizontal scale + HA across AZs; **the DB is usually the bottleneck** (cache, read replicas, sharding, async writes); cache trade-off = speed vs staleness; CAP = under partition pick C or A (web apps usually AP).

### Block 4 — System design practice (highest ROI) · 2.5h · `T-14.7h → T-12.2h`

Design each out loud, on paper/whiteboard, using the framework. These are copied from your JD — expect them. For each, answer: Why this architecture? What if traffic 10×? How do you monitor it? How do you recover from failure?

- [ ] **CI/CD pipeline** (Dev → Git → PR → build → tests → Docker → security scan → ECR → Terraform → deploy → monitoring → rollback).
- [ ] **Internal Developer Platform** (Developer → Portal → template → Terraform → AWS → GitOps → deploy → logs). *This is literally your job — nail it.*
- [ ] **Log aggregation / monitoring system** (agents → queue/Kafka → processors → Elasticsearch/S3 → Grafana/Kibana → alerting; trade-offs: retention vs cost, sampling, indexing).
- [ ] **AI inference infrastructure / your Docker Optimizer** (architecture, flow, storage, API, caching, scaling, failure handling).
- [ ] Rehearsed the pick-your-design move: **steer to infra/pipeline or logging/monitoring** where your DevOps depth shows.

### Block 5 — Project deep-dives · 1.5h · `T-12.2h → T-10.7h`

Reference: [13-project-related](./13-project-related). For each project prepare: problem statement, architecture, tech used, challenges, trade-offs, failure handling, what you'd improve. Likely prompts: "Walk me through the IDP." "How does Terraform fit in?" "How do developers deploy?" "How do you prevent config drift?" "How would you scale to hundreds of teams?"

- [ ] Internal Developer Platform
- [ ] ECS Terraform module
- [ ] Docker Optimizer
- [ ] Bedrock integration
- [ ] Lambda automation
- [ ] **Exit check:** can talk about the IDP naturally for ~10 minutes.

### Block 6 — Out-loud mock · 1h · `T-10.7h → T-9.7h`

Stand up, speak, ideally record yourself:

- [ ] Write Two Sum
- [ ] Design a CI/CD pipeline
- [ ] Explain GitOps
- [ ] Design a deployment platform
- [ ] Explain your Docker Optimizer

### Sleep · 7-8h · `T-9.7h → T-1.7h`

- [ ] 7-8h sleep — non-negotiable. A rested brain beats one more problem.

### Morning revision · 1.5h · `T-1.7h → T-0`

- [ ] Re-read R2 cram **"7 things to memorize."**
- [ ] Redo **3** coding problems from memory (Two Sum, Valid Parentheses, First Unique).
- [ ] Re-explain **1** design (CI/CD) aloud in <5 min.
- [ ] Re-read the 5-step answer pattern + honesty scripts.

---

## 4. WINDOW 2 — 20 hours to Round 2 (Cloud + IaC + GitOps)

This round is your strength. Target ~8.5h focused + break + sleep + morning revision. Primary doc all window: [docusign-cloud-devops-1hr-cram.md](./docusign-cloud-devops-1hr-cram.md).

### Block 1 — AWS: networking + IAM + IRSA · 2h · `T-20h → T-18h`

References: [10-aws](./10-aws) (full note set), [04-networking](./04-networking), 1hr cram **Section 1**.

- [ ] Networking fluent: VPC, subnets (public vs private), route tables, IGW, NAT Gateway — [10-aws/01-vpc-subnets-routing.md](./10-aws/01-vpc-subnets-routing.md).
- [ ] **Security Groups vs NACLs** (stateful vs stateless) — can explain cold — [10-aws/02-security-groups-vs-nacls.md](./10-aws/02-security-groups-vs-nacls.md).
- [ ] IAM user/role/policy + trust policy + policy evaluation (explicit deny wins) — [10-aws/03-iam-users-roles-policies.md](./10-aws/03-iam-users-roles-policies.md).
- [ ] **IRSA** — how an EKS pod gets AWS creds via OIDC + ServiceAccount → IAM role — [10-aws/04-irsa.md](./10-aws/04-irsa.md).
- [ ] ECS vs EKS / why EKS — [10-aws/05-ecs-vs-eks.md](./10-aws/05-ecs-vs-eks.md).
- [ ] Name comfortably: EC2, S3, ALB/NLB, Route 53, ECR, CloudWatch, RDS, Secrets Manager, Parameter Store, Auto Scaling — [10-aws/08-core-services-secrets-and-cost.md](./10-aws/08-core-services-secrets-and-cost.md).
- [ ] Scenario: app can't reach DB (SG rules, subnet routing).
- [ ] Scenario: ALB returns 502 — [10-aws/06-load-balancers-and-502.md](./10-aws/06-load-balancers-and-502.md) + [04-networking/06-502-bad-gateway.md](./04-networking/06-502-bad-gateway.md).
- [ ] Scenario: IAM AccessDenied — where do you check? — [10-aws/09-iam-access-denied-troubleshooting.md](./10-aws/09-iam-access-denied-troubleshooting.md).
- [ ] Scenario: traffic 10× — what changes? — [10-aws/07-high-availability-and-autoscaling.md](./10-aws/07-high-availability-and-autoscaling.md).
- [ ] **Service communication** (how resources talk inside/across VPC, subnet, AZ, account; where SG applies vs IAM-only services like S3/DynamoDB; EC2→S3 needs path + IAM) — [10-aws/10-service-communication.md](./10-aws/10-service-communication.md).
- [ ] Compute options **EC2 vs Lambda vs ECS vs EKS** (control-vs-ops spectrum, Fargate) — [10-aws/05-ecs-vs-eks.md](./10-aws/05-ecs-vs-eks.md).

### Block 2 — Terraform · 2h · `T-18h → T-16h`

References: [06-terraform](./06-terraform), 1hr cram **Section 2**.

- [ ] State + remote backend (S3 + DynamoDB lock) + locking.
- [ ] Modules — inputs/outputs, versioning ([06-terraform/02-terraform-modules.md](./06-terraform/02-terraform-modules.md)).
- [ ] Variables/outputs, workspaces, `lifecycle`, `depends_on`, data sources, `for_each` vs `count` ([06-terraform/01-for_each-vs-for.md](./06-terraform/01-for_each-vs-for.md)).
- [ ] Scenario: `plan` wants to recreate prod resources — what do you do?
- [ ] Scenario: two engineers apply simultaneously (locking).
- [ ] Scenario: state drift; import existing resources; organizing multi-environment — deep-dive: [06-terraform/03-import-blocks-and-state-recovery.md](./06-terraform/03-import-blocks-and-state-recovery.md) (import blocks, `-generate-config-out`, and recovering a lost 1000-resource state file).

### Block 3 — Kubernetes · 2h · `T-16h → T-14h`

Reference: [08-kubernetes](./08-kubernetes). **Start here:** [08-kubernetes/00-kubernetes-one-shot-handbook.md](./08-kubernetes/00-kubernetes-one-shot-handbook.md) — the one-shot flow that ties every K8s concept together (Pod → Deployment → Service → Ingress → config → storage → probes → scaling → security → GitOps), with industry-usage notes and a cheat table.

- [ ] Core objects: Pod, Deployment, ReplicaSet, ConfigMap/Secret, Namespace.
- [ ] Service types ([08-kubernetes/05-types-of-services.md](./08-kubernetes/05-types-of-services.md)) + Ingress vs Ingress Controller ([08-kubernetes/23-ingress-controller-vs-ingress.md](./08-kubernetes/23-ingress-controller-vs-ingress.md)).
- [ ] Liveness vs readiness ([08-kubernetes/20-liveness-vs-readiness.md](./08-kubernetes/20-liveness-vs-readiness.md)), HPA, requests/limits, network policies ([08-kubernetes/12-network-policies.md](./08-kubernetes/12-network-policies.md)).
- [ ] Scenario: CrashLoopBackOff debug ([08-kubernetes/19-crashloopbackoff.md](./08-kubernetes/19-crashloopbackoff.md)) + ImagePullBackOff.
- [ ] Scenario: "pods Running but app unreachable" / users get 502s — where to look.
- [ ] Scenario: rolling updates / zero-downtime ([08-kubernetes/16-all-deployment-strategies.md](./08-kubernetes/16-all-deployment-strategies.md)).

### Break · 20m · `T-14h → T-13.7h`

### Block 4 — GitOps & GitHub Actions · 1.5h · `T-13.7h → T-12.2h`

Reference: [05-cicd](./05-cicd) (esp. [05-cicd/14-argocd-app-out-of-sync.md](./05-cicd/14-argocd-app-out-of-sync.md)).

- [ ] GitOps workflow + Argo CD sync + drift detection + rollback (revert the Git commit).
- [ ] GitHub Actions: PR workflow, secrets handling, runners/artifacts/environment approvals.
- [ ] Know the **GitOps vs CI/CD** difference.
- [ ] **Marquee question rehearsed cold:** *"A developer merges a PR — walk me through every step until it's live in production."* (Git → CI build+test → Docker → ECR → update manifest repo → Argo CD reconciles → EKS rollout → observability → rollback path.)

### Block 5 — Observability · 45m · `T-12.2h → T-11.5h`

Reference: [09-observability](./09-observability).

- [ ] Metrics / logs / traces (the three pillars) + Prometheus, Grafana, CloudWatch, alerting.
- [ ] Scenario: *"The application is slow — how do you investigate?"* (metrics → narrow tier → logs/traces → resource limits → dependencies).

### Block 6 — Scenario mock + project tie-in · 1h · `T-11.5h → T-10.5h`

Out loud, tying each back to your IDP/ECS module work:

- [ ] "developer merges PR → prod" full walk-through
- [ ] Debug a CrashLoopBackOff
- [ ] Terraform plan wants to recreate prod
- [ ] IAM AccessDenied triage

### Sleep · 7-8h · `T-10.5h → T-2.5h`

- [ ] 7-8h sleep.

### Morning revision · 1.5h · `T-2.5h → T-0`

- [ ] Read [docusign-cloud-devops-1hr-cram.md](./docusign-cloud-devops-1hr-cram.md) top to bottom.
- [ ] Say the scenario one-liners aloud: SG vs NACL, IRSA, TF state/locking/drift, PR→prod flow, ECS vs EKS, "app is slow" triage.

---

## 5. Appendix A — Question banks

### Coding (Round 1)

Two Sum · Valid Anagram · Contains Duplicate · Best Time to Buy/Sell Stock · Merge Sorted Array · Valid Parentheses · Reverse Linked List · Middle of Linked List · Binary Search · First Unique Character · Frequency counter. DevOps variants: parse logs, count HTTP status codes, find duplicate lines, read CSV/JSON, call an API, process files.

### System Design (Round 1)

Design a CI/CD pipeline · Design a deployment platform · Design an Internal Developer Portal · Design a log aggregation / monitoring system · Design an AI inference platform (your Docker Optimizer) · Design a scalable web application · (stretch) URL shortener, notification system, file storage.

### Cloud / AWS scenarios (Round 2)

Explain VPC · debug networking · IAM permission denied (where to check) · ECS vs EKS · high availability · cost optimization · traffic increases 10× (what changes) · app can't reach DB · ALB 502.

### Terraform scenarios (Round 2)

Remote state · modules · drift · safe production changes · importing existing resources · `plan` wants to recreate prod · two engineers apply at once (locking) · multi-environment layout.

### Kubernetes scenarios (Round 2)

Pod lifecycle · debug CrashLoopBackOff / ImagePullBackOff · "pods Running but app unreachable" · Services / Ingress · network policies · rolling updates / zero-downtime.

### GitOps / Observability scenarios (Round 2)

PR → production full flow · Argo CD sync & drift · rollbacks · secrets in pipelines · "application is slow" triage.

---

## 6. Appendix B — Curated resources (avoid rabbit holes)

- **Coding:** LeetCode — Easy list only.
- **System Design (overview videos, don't binge):** ByteByteGo (YouTube), Gaurav Sen (YouTube).
- **AWS:** AWS Skill Builder (architecture diagrams) + AWS docs for services you already use.
- **Terraform:** Official HashiCorp Learn (modules, state, backends).
- **Kubernetes:** Official Kubernetes Concepts docs.
- **GitOps:** Argo CD docs (architecture + sync flow).
- **FAQ / question sweeps (skim as checklists only):**
  - Kubeify — Top 50 AWS Cloud Platform Engineering Questions: [https://kubeify.com/blog/top-50-aws-cloud-platform-engineering-questions-answers](https://kubeify.com/blog/top-50-aws-cloud-platform-engineering-questions-answers)
  - Glassdoor — Cloud/Platform Engineer interview reports: [https://www.glassdoor.co.in/Interview/hyderabad-sr-cloud-platform-engineer-interview-questions-SRCH_IL.0%2C9_KO10%2C36.htm](https://www.glassdoor.co.in/Interview/hyderabad-sr-cloud-platform-engineer-interview-questions-SRCH_IL.0%2C9_KO10%2C36.htm)
  - Real DevOps/Cloud interview questions (scenario-based): [https://proxytechsupport.com/interviews/2026-04-25-real-devops-cloud-engineer-interview-questions-aws-kubernetes-helm-cicd-gitops/](https://proxytechsupport.com/interviews/2026-04-25-real-devops-cloud-engineer-interview-questions-aws-kubernetes-helm-cicd-gitops/)
  - GitHub — DevOps interview questions repo: [https://github.com/immanuwell/DevOps-interview-questions](https://github.com/immanuwell/DevOps-interview-questions)

Guidance: **explain concepts aloud** beats passive reading. Use videos/docs only to reinforce structure, then close them and re-explain from memory.

---

## 7. Appendix C — Must-memorize cheat block

### Round 1 — the 7 things (from R2 cram)

1. **Hash map kills nested loops** → O(n²) becomes O(n).
2. **Brute force → optimize → complexity**, always narrated.
3. **Stateless app tier + load balancer** = horizontal scaling + HA across AZs.
4. **The DB is usually the bottleneck** → cache, read replicas, sharding, async writes.
5. **Cache trade-off** = speed vs staleness/invalidation.
6. **CAP** = under partition, pick Consistency or Availability; web apps usually AP.
7. **Design framework** = clarify → scale numbers → diagram → bottlenecks → trade-offs.

### Round 2 — the 5 things

1. **Security Group vs NACL** — SG = instance-level, stateful, allow-only; NACL = subnet-level, stateless, allow+deny.
2. **IRSA** — EKS pod → ServiceAccount → IAM role via OIDC; no keys in the pod.
3. **Terraform state** — remote backend (S3) + DynamoDB locking; `plan` detects drift; `import` for existing resources.
4. **PR → production (GitOps)** — Git → CI (build/test/Docker/ECR) → update manifest repo → Argo CD reconciles → EKS → observe → rollback = revert commit.
5. **ECS vs EKS** — ECS simpler/AWS-native; EKS = Kubernetes (portable, richer ecosystem, more ops overhead).

---

*Do the reps out loud. In these rounds, how you reason and communicate matters as much as the answer. Good luck.*