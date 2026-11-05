## CI/CD From Scratch + GitOps — The Deep-Dive Every Interviewer Drills

### Question  
"We've given you a project — **configure CI/CD from scratch.** Walk me through the whole GitOps process, how it works in your org, end to end." Then the drill-down on **every** step: *"How does the checkout happen? How does the pipeline get access to GitHub? How does it know where the code lives? How does it push to ECR? How does Argo CD deploy it? And at each step — how is it **authenticated and authorized**? How is it secure?"*

### 📝 Short Explanation  
This is the most-asked pipeline question, and it's really a test of **whether you understand each component *interaction*** — not just the happy-path stages. The winning move: narrate the flow **stage by stage**, and at every hop where two systems talk, answer with the [**authN → authZ** framework](../10-aws/29-authentication-vs-authorization.md): *who is this (authentication)* and *what may it do (authorization)*, plus how the secret/identity is delivered. Do that and the drill-down has nowhere to go.

## ✅ Answer  
- **CI (Jenkins/GitHub Actions):** a **webhook** (verified with a shared secret) triggers the pipeline → **checkout** (via a GitHub App/deploy key/PAT) → lint/test/**SAST** → **build image** → **image scan** → **push to ECR** (auth via **IAM/OIDC**, not stored keys).
- **CD (GitOps/Argo CD):** CI **updates the image tag in a Git manifests repo** → **Argo CD** (watching Git) syncs the change to the cluster → Kubernetes **pulls the image from ECR** → rolls out with probes → **Service/Ingress** exposes it.
- **Security at every hop = authN + authZ:** each system→system call authenticates with a **short-lived, scoped identity** (OIDC/IRSA/tokens) and is authorized by a **least-privilege policy** (IAM/RBAC/repo scope). No long-lived secrets, immutable image tags, and a **deployment gate** before prod.

## 📘 Detailed Explanation

### The end-to-end flow
```text
Developer ──merge──► GitHub (main)
   │  (1) webhook (HMAC-signed) 
   ▼
Jenkins CI  (2) checkout  (3) lint/test/SAST  (4) build image
   │         (5) image scan  (6) push to ECR   [(7) deployment gate]
   ▼
ECR (immutable image :git-sha / @sha256)
   │  (8) CI commits new image tag to the MANIFESTS repo in Git
   ▼
Git (manifests) = desired state
   │  (9) Argo CD detects the commit
   ▼
Argo CD ──(10) sync──► Kubernetes API
   │  (11) kubelet pulls image from ECR
   ▼
Pods ─► Service ─► Ingress/ALB ─► Users
   (throughout: least privilege, secrets mgmt, scanning, logging, rollback)
```

### Stage by stage — and *how each hop is authenticated & authorized*

**1. Trigger — GitHub webhook → Jenkins**
- **How it works:** on merge to `main`, GitHub POSTs a webhook to Jenkins; the `githubPush()` trigger starts the job. (Alternative: Jenkins SCM polling, but webhooks are push-based and instant.)
- **AuthN:** the webhook is signed with a **shared secret** (HMAC `X-Hub-Signature-256`); Jenkins verifies it so a random caller can't trigger builds.
- **AuthZ:** the Jenkins endpoint is restricted (network/allow-list) and the job is scoped to that repo.
- *Interview line:* "The webhook is the **event**; the Jenkins **trigger** is what reacts to it. I verify the webhook's HMAC signature so only GitHub can start the pipeline."

**2. Checkout — "how does the pipeline get access to GitHub / know where the code lives?"**
- **How it works:** the pipeline knows the repo from its **SCM config** (the Git URL in the job / `checkout scm`). It clones the **exact commit SHA** that triggered the build (not just "latest") for reproducibility.
- **AuthN (the crux they drill):** Jenkins authenticates to GitHub with a **credential** — best is a **GitHub App** (short-lived installation token) or a **deploy key (SSH)** scoped to that one repo; a **fine-grained PAT** is the fallback. Stored in the **Jenkins credentials store**, never in the Jenkinsfile.
- **AuthZ:** that credential has **read scope** on exactly that repo (deploy key = single repo; GitHub App = least-privilege permissions).
- *Interview line:* "It knows where the code lives from the job's SCM/remote URL, and it authenticates with a repo-scoped **GitHub App/deploy key** — read-only, one repo, pulled from the credentials store."

**3. Validate — lint, unit tests, dependency/SAST scan**
- Lint (`flake8`), tests (`pytest`), dependency audit (`pip-audit`), and **SAST** (Semgrep/SonarQube). Fail fast on any gate.

**4. Build the image**
- Multi-stage `Dockerfile`, install from `requirements.txt`, tag with the **git commit SHA** (immutable), run as **non-root**, minimal base image. In-cluster builds use **Kaniko/BuildKit** (no privileged Docker daemon).

**5. Image scan**
- **Trivy/Grype** scan for CVEs and misconfig; **fail the build on criticals**. Also verify no secrets baked in.

**6. Push to ECR — "how does Jenkins get access to AWS?"**
- **How it works:** authenticate the Docker client to ECR, then push `:<git-sha>`.
- **AuthN (drilled):** the runner assumes an **IAM role** — via **IRSA** (Jenkins on EKS) or **GitHub OIDC** (GitHub Actions) — and gets **temporary STS credentials**. `aws ecr get-login-password` returns a short-lived token for `docker login`. **No static AWS keys stored.**
- **AuthZ:** that role's policy allows only **`ecr:PutImage` / upload** on the specific repo (least privilege) — it can't touch other AWS resources.
- *Interview line:* "The agent has **no stored AWS keys** — it assumes a role via **OIDC/IRSA**, gets temporary creds, and its policy only permits pushing to that one ECR repo."

**7. Deployment gate (before it can reach the cluster)**
- Policy checks: **no critical CVEs**, image came from the **trusted ECR repo**, tag is **immutable** (SHA/digest, never `latest`), and required **approvals** for staging/prod. Dev can auto-promote on a clean scan; prod needs stricter gates.

**8. Update the manifest in Git (CI → CD handoff)**
- CI edits the **image tag** in the **manifests repo** (Helm/Kustomize) and commits/opens a PR. **Git is now the desired state / source of truth.** This is the CI↔CD boundary.
- **AuthN/AuthZ:** CI writes with a **narrowly-scoped token** that can push only to the manifests repo.

**9–10. Argo CD detects & syncs (the CD engine)**
- **How it works:** Argo CD **watches the Git repo**, sees the new commit, diffs desired (Git) vs live (cluster), and **syncs** (auto-sync + self-heal). *(Key point interviewers love: Argo CD watches **Git, not ECR** — see [note 16](./16-argocd-gitops-and-image-updates.md).)*
- **AuthN — Argo CD → Git:** a **repo deploy key/token** (read-only on manifests).
- **AuthN — Argo CD → Kubernetes API:** Argo CD runs with a **ServiceAccount**; its **token** authenticates to the API server.
- **AuthZ — in the cluster:** **RBAC** limits that ServiceAccount to the namespaces/resources it may apply. Argo CD **Projects** further restrict which repos/clusters/namespaces an app can touch.

**11. Kubernetes pulls the image & rolls out**
- **AuthN — cluster → ECR:** the node/pod pulls via **`imagePullSecrets`** or, cleanly, the **node role / IRSA** with `ecr:GetDownloadUrlForLayer` etc.
- Rollout uses **readiness/liveness probes**, `RollingUpdate` (or canary), and a **PodDisruptionBudget**; **Service** gives stable networking, **Ingress/ALB** exposes it with **TLS**.

**12. Pod → AWS at runtime**
- The app gets AWS access via **IRSA** (ServiceAccount wears an IAM role) — **not** baked-in keys. AuthN = the projected token → `AssumeRoleWithWebIdentity`; AuthZ = the role's least-privilege policy.

### A representative Jenkinsfile (Python) — with the ECR auth the drill-down expects
```groovy
pipeline {
  agent { label 'docker-agent' }            // where it runs (see agents below)
  environment {
    AWS_REGION = 'us-east-1'
    ECR = '123456789012.dkr.ecr.us-east-1.amazonaws.com/my-python-app'
    TAG = "${GIT_COMMIT}"                    // immutable: commit SHA, not 'latest'
  }
  triggers { githubPush() }                  // reacts to the verified webhook
  stages {
    stage('Checkout')  { steps { checkout scm } }                         // repo-scoped credential
    stage('Test/Lint') { steps { sh 'pip install -r requirements.txt && flake8 . && pytest' } }
    stage('SAST/Deps') { steps { sh 'pip-audit' } }
    stage('Build')     { steps { sh 'docker build -t $ECR:$TAG .' } }
    stage('Image Scan'){ steps { sh 'trivy image --exit-code 1 --severity CRITICAL $ECR:$TAG' } }
    stage('Login+Push'){ steps {                                          // <-- the ECR auth step
        sh 'aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR'
        sh 'docker push $ECR:$TAG'
    } }
    stage('Bump Manifest') { steps {          // CI updates the Git manifest tag (GitOps handoff)
        sh './scripts/set-image-tag.sh $ECR:$TAG && git commit -am "deploy $TAG" && git push'
    } }
  }
  post { failure { echo 'pipeline failed' } }
}
```
*(Note: the agent has no AWS keys — `aws` uses the role from IRSA/OIDC. That's the answer to "how does it get access to ECR?")*

### `agent` and `trigger` (they always ask)
- **`agent`** = *where* the pipeline runs. `agent any` = any available executor — fine when the job needs no special tooling. Use a **labelled agent** (`agent { label 'docker-agent' }`) or an **EC2/Kubernetes agent** when you need specific tooling (Docker/BuildKit), a specific runtime version, GPU, or **network access to private resources** (e.g. a VPC-only ECR/EKS). *"`agent any` when the environment doesn't matter; a labelled EC2/K8s agent when I need specific tools or VPC network access."*
- **`trigger`** = *when/why* the pipeline starts (`githubPush()`, `cron`, upstream job). The webhook is the event; the trigger is the config that reacts. *"Trigger = when it runs; agent = where it runs."*

### Where authN + authZ live at every hop (summary table)
| Hop | AuthN (who) | AuthZ (what) |
|---|---|---|
| GitHub → Jenkins (webhook) | HMAC shared secret | restricted endpoint |
| Jenkins → GitHub (checkout) | GitHub App / deploy key / PAT | repo read scope |
| Jenkins → ECR (push) | **IRSA/OIDC → STS temp creds** | `ecr:PutImage` on one repo |
| Argo CD → Git | deploy key / token | manifests read |
| Argo CD → K8s API | ServiceAccount token | **RBAC** + Argo Project |
| K8s → ECR (pull) | node role / IRSA / imagePullSecret | `ecr:Get*` |
| Pod → AWS | **IRSA** token | role's least-privilege policy |

### Secrets, promotion, rollback (name these)
- **Secrets:** app secrets from **Secrets Manager/Vault** (via External Secrets Operator), never in the image or Git; config in ConfigMaps.
- **Promotion:** promote the **same immutable image** dev → stage → prod (don't rebuild per env); env differences live in Kustomize overlays.
- **Rollback:** **revert the Git commit** (image tag) → Argo CD reconciles the previous version. GitOps makes rollback a `git revert`.

## Summary
- Narrate **CI (webhook→checkout→test/scan→build→scan→push ECR→gate) then CD (update Git tag→Argo CD sync→K8s pull→rollout→expose)**.
- At **every component hop**, answer with **authN (who — short-lived scoped identity: OIDC/IRSA/tokens) + authZ (what — least-privilege IAM/RBAC/repo scope)**. That's exactly what the drill-down is testing.
- Keys facts that stop the drill: **checkout uses a repo-scoped GitHub App/deploy key**, **ECR push uses IRSA/OIDC temp creds (no stored keys)**, **Argo CD watches Git not ECR**, **images are immutable SHAs**, and **rollback = revert the Git commit**.

## Related
- [Authentication vs Authorization framework](../10-aws/29-authentication-vs-authorization.md) — the lens for every hop
- [Argo CD, GitOps & image updates](./16-argocd-gitops-and-image-updates.md) · [App OutOfSync](./14-argocd-app-out-of-sync.md)
- [GitHub OIDC → AWS](../10-aws/26-github-actions-oidc-federation.md) · [IRSA](../10-aws/104-irsa.md) · [Kubernetes RBAC](../08-kubernetes/36-rbac.md)
- [Jenkins shared libraries](./01-jenkins-shared-libraries.md)
