## Jenkins: Freestyle vs Pipeline Jobs, and Managing Credentials

### Question  
What's the difference between a **Freestyle** job and a **Pipeline** job in Jenkins? And how do you **store and use credentials** securely in Jenkins?

### 📝 Short Explanation  
Two everyday Jenkins questions. **Freestyle = UI-configured, single job**; **Pipeline = code (`Jenkinsfile`) in the repo**, which is what you want for real CI/CD (versioned, reviewable, multi-stage). Credentials should live in the **Jenkins Credentials store** and be injected at runtime — **never** hardcoded in the job or `Jenkinsfile`.

## ✅ Answer  
- **Freestyle job**: configured through the **UI**, good for simple one-off tasks; logic lives in Jenkins, not in Git, and it doesn't express multi-stage flows well.
- **Pipeline job**: defined as code in a **`Jenkinsfile`** (declarative or scripted), **version-controlled** with the app, supports **stages, parallelism, agents, shared libraries**, and survives controller restarts. This is the standard for production CI/CD.
- **Credentials**: store them in the **Jenkins Credentials store** (scoped by folder/domain), reference by **ID**, inject with `withCredentials`/`credentials()`, and mask them in logs. Better still, use **short-lived identity** (AWS **OIDC/IRSA**) so there's no long-lived secret at all.

## 📘 Detailed Explanation  

### Freestyle vs Pipeline
| | Freestyle | Pipeline |
|---|---|---|
| Defined in | **UI** (point-and-click) | **`Jenkinsfile`** (code, in Git) |
| Version control | No (config lives in Jenkins) | **Yes** (reviewed via PR) |
| Multi-stage | Clunky | **Native `stages{}`** |
| Parallelism / agents per stage | Limited | **Yes** |
| Reuse | Copy jobs | **Shared libraries** |
| Restart resilience | — | **Durable** (resumes) |
| Best for | Trivial/legacy tasks | **Real CI/CD** |

**One-liner:** *"Freestyle is UI-configured and fine for simple tasks; Pipeline is code-as-configuration in a `Jenkinsfile` — versioned, reviewable, multi-stage — which is what I use for production."*

Declarative (`pipeline { agent … stages { … } }`) is the readable default; scripted (Groovy) is more flexible for complex logic.

### Managing credentials in Jenkins
**The rule: secrets never live in the job config or the `Jenkinsfile`.** They live in the **Credentials store** and are pulled in at runtime.
- **Types:** username/password, secret text (tokens), **SSH keys**, secret files, and cloud (e.g. AWS keys — though prefer roles).
- **Scope:** system, global, or **folder/domain-scoped** (least privilege — a team's jobs only see their creds).
- **Reference by ID and inject:**
```groovy
stage('Deploy') {
  steps {
    withCredentials([usernamePassword(credentialsId: 'ecr-bot',
                     usernameVariable: 'U', passwordVariable: 'P')]) {
      sh 'echo "$P" | docker login -u "$U" --password-stdin $ECR'
    }
  }
}
// or, environment binding:
environment { GH_TOKEN = credentials('github-app-token') }
```
- **Masking:** Jenkins masks credential values in console output; still avoid `echo`ing them.
- **Plugins/patterns:** the **Credentials Binding** plugin, **HashiCorp Vault** plugin for dynamic secrets, and the **AWS Credentials/OIDC** approach.

### The better answer: no long-lived secrets
> "Where I can, I avoid storing cloud secrets entirely — the Jenkins agent **assumes an AWS role via OIDC/IRSA** and gets **temporary credentials**, so there's nothing to leak or rotate. For things that must be static (a bot token), I keep them in the **Credentials store** (folder-scoped) or pull from **Vault**, and inject with `withCredentials`."

### ⚠️ Things they probe
- **Pipeline (as code) over Freestyle** for anything real, and *why* (versioned, reviewable, multi-stage, durable).
- **Never hardcode secrets**; use the **Credentials store + `withCredentials`**, scoped least-privilege.
- Bonus points: **OIDC/IRSA for temporary creds** and **Vault** for dynamic secrets.

---

### 🧠 Real-world Example

> "We moved legacy **Freestyle** jobs to **`Jenkinsfile`** pipelines in each repo, so builds were reviewed in PRs and used **shared libraries** for common stages. Cloud access used **OIDC to assume an AWS role** (no stored keys); the few static tokens lived in **folder-scoped credentials** and were injected with `withCredentials`, never printed."

> Summary:  
> **Freestyle = UI, simple, not in Git; Pipeline = `Jenkinsfile` as code — versioned, multi-stage, durable (use it for real CI/CD).** Keep secrets in the **Credentials store** (scoped), inject with `withCredentials`/`credentials()`, and prefer **OIDC/IRSA temporary creds** or **Vault** over long-lived secrets.

### Related
- [CI/CD from scratch + GitOps deep dive](./17-cicd-from-scratch-gitops-deep-dive.md) · [Jenkins shared libraries](./01-jenkins-shared-libraries.md)
- [GitHub OIDC → AWS](../10-aws/26-github-actions-oidc-federation.md) · [Vault — secrets management](../10-aws/119-vault-secrets-management.md)
