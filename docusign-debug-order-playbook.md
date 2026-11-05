<!-- #genai: cross-cutting debug-order playbook (AWS, Kubernetes, CI/CD, app performance) with one memorable framework, for Docusign interview prep -->

# Debug-Order Playbook

One place for "in what order do I debug X?" Interviewers love **calm, systematic ordering** more than the
right guess. Learn the **one universal framework** first, then the per-scenario checklists that specialize it.

---

## The universal framework (memorize this one thing)

**I-N-C-A-D** — work from the outside in, verify each layer before the next:

1. **Identity** — *who* is making the call, and is it who I think? (`get-caller-identity`, which SA/role/user)
2. **Network** — is there a **path** and do **firewalls** allow it? (route, SG, NACL, DNS, port)
3. **Config** — env vars, ConfigMaps/Secrets, manifests, pipeline vars — is the config correct?
4. **App** — read the **actual error/logs** of the thing that failed (not the first log — the *real* error).
5. **Dependencies** — downstream DB / API / registry / KMS / external service.

Plus three rules that matter more than the order:
- **Read the real error message** — don't guess; the error usually names the action/resource/reason.
- **Change one thing at a time** — otherwise you can't tell what fixed it.
- **Don't stop at the first cause** — check auth, network, config, dependency, environment.

---

## AWS — "resource A can't reach/act on resource B"
Remember: every AWS call = **network path AND IAM**. Decide which layer first.

1. **Identity:** `aws sts get-caller-identity` — is the caller the role/user you expect? (On EKS: is IRSA wired?)
2. **DNS/endpoint:** does the name resolve? Right region/endpoint?
3. **Route:** is there a route to the target? (`local` for in-VPC; NAT/IGW/**VPC endpoint** for AWS services)
4. **Security Group:** does the target's SG allow the source (ideally by **SG reference**) on the right port?
5. **NACL:** subnet allows **both** directions (stateless → return traffic too)?
6. **IAM (for API services):** identity policy allows the `Action`+`Resource`? any **explicit Deny**? **SCP**/boundary? resource policy (bucket/KMS)? `kms:Decrypt`?
7. **Tools:** CloudTrail (the exact denied call), VPC Flow Logs (dropped packets), IAM Policy Simulator.

## AWS — IAM `AccessDenied`
1. `get-caller-identity` (verify the real principal).
2. Read the denial — it names the action + resource.
3. Identity **permission policy** — is it allowed?
4. **Explicit Deny** anywhere? (deny beats allow)
5. **SCP / permission boundary / session policy**?
6. **Resource-based policy** (S3 bucket, KMS key) + `Condition` (MFA/IP/VPC)?
7. **KMS**: encrypted resource also needs `kms:Decrypt`.

## AWS — ALB 502 / 503
- **502** = target returned a bad/failed response. **503** = no healthy targets.
1. **Target group health** — targets healthy?
2. **App logs** on the target — crashing / 5xx / OOM?
3. **Timeouts** — app slower than ALB idle timeout / keep-alive mismatch (classic 502).
4. **Security groups** — target SG allows the ALB SG on the app port?
5. **Port/protocol** — target-group port vs actual listen port; HTTP vs HTTPS.
6. **ALB metrics** — `HTTPCode_ELB_5XX` (ALB side) vs `HTTPCode_Target_5XX` (app side).

---

## Kubernetes — general pod problem
Order: **events → logs → config → resources → networking.**
1. `kubectl describe pod` — read **Events** at the bottom first (scheduling, pull, probe failures).
2. `kubectl logs <pod>` and `kubectl logs <pod> --previous` (previous crash).
3. **Config** — ConfigMap/Secret present and mounted? env correct?
4. **Resources** — requests/limits; OOMKilled? node capacity?
5. **Networking** — Service endpoints exist? selector matches labels? readiness passing? `targetPort` right?

### CrashLoopBackOff
`logs --previous` → bad config/missing env/secret → OOM (raise limits) → failing dependency at startup.

### ImagePullBackOff
`describe` events → wrong image name/tag → **registry/ECR auth** (node role or pull secret) → network to registry.

### Pending
`describe` events → **no capacity** (requests too big) → **taints/affinity/selectors** unschedulable → **PVC** unbound.

### Service reachable internally but not externally / 502 via Ingress
Endpoints populated? → selector matches pod labels? → **readiness** passing (unready pods are pulled)? → `targetPort` matches container port? → **Ingress controller** healthy? → **NetworkPolicy** blocking?

---

## CI/CD pipeline failure
1. **Read the log — find the *last* error**, not the first (the first is often a downstream symptom).
2. **Recent changes** — what changed since the last green run? (`git diff` / changeset / SCM).
3. **Reproduce locally** — same command; "works on my machine" ⇒ **environment difference**.
4. **Dependencies** — registry/proxy reachable? version pinned? **cache** stale or poisoned?
5. **Credentials/secrets** — expired token, missing secret, wrong scope/OIDC role.
6. **Environment** — runner image, tool versions, env vars, resource limits (OOM in build).
7. **Flaky?** — re-run once; if intermittent, suspect timeouts, ordering, or shared state.

### "CI passed but the app is broken in prod"
Config/env differs (prod vs CI) → secrets/feature flags → migration not run → image tag/artifact mismatch → infra (SG/IAM/DNS) differs → smoke tests missing.

---

## App is slow (perf triage)
1. **Metrics first** — which **tier** is slow? (LB, app, DB, dependency) via dashboards.
2. **Narrow the layer**, then read **logs/traces** for that tier.
3. **Resources** — CPU/memory/throttling; hitting limits/HPA ceiling?
4. **Database** — slow queries, missing index, connection-pool exhaustion, lock contention.
5. **Dependencies** — a downstream API/queue latency bleeding upstream.
6. **Recent change** — deploy/config that correlates with the regression.

---

## The one-liners to say in an interview
- **AWS connectivity:** *"Every call is network path plus IAM — I check identity, then route/SG/NACL/endpoint, then IAM policies and resource/KMS policies."*
- **Kubernetes:** *"`describe` for events, then `logs --previous`, then config, then resources, then Service endpoints/readiness."*
- **ALB 502:** *"502 means the target failed — I check target health, app logs, timeouts, SG, then port/protocol."*
- **CI/CD:** *"Read the last error, look at what changed, reproduce locally, then check deps, creds, and environment."*
- **Golden rules:** *"Read the real error, change one thing at a time, and don't stop at the first cause."*
