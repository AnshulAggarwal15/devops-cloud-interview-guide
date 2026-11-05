## Argo CD, GitOps & How a New Image Actually Gets Deployed

### Question  
What do you actually *do* with Argo CD? How is it integrated, and what's the deployment pipeline? If you push a new image (say `3.5`) to **ECR** while `3.4` is running, **how does Argo CD know** there's a new image and deploy it?

### 📝 Short Explanation  
The trap in this question: Argo CD is a **GitOps** tool — by default it watches a **Git repository of manifests**, **not** your container registry. So pushing a new image to ECR **does not** trigger Argo CD on its own. Something has to **update the image tag in Git** (your CI, or Argo CD Image Updater). If you don't know that, the follow-ups expose it.

## ✅ Answer  
- Argo CD **continuously syncs** the live cluster state to the **desired state declared in Git** (manifests / Helm / Kustomize). It detects drift, shows sync status, and can **auto-sync + self-heal**.
- It watches **Git, not ECR.** So the standard flow is: **CI builds the image → pushes to ECR → CI updates the image tag in the Git manifests repo → Argo CD sees the Git commit → syncs → deploys.**
- To deploy on a **new image** without a manual Git edit, either your **CI commits the new tag**, or you run **Argo CD Image Updater**, which polls the registry and writes the new tag back to Git.

## 📘 Detailed Explanation

### What Argo CD does (the "what do you do with it" answer)
- Define an **`Application`** pointing at a Git repo/path/revision and a target cluster/namespace.
- Argo CD renders the manifests (plain YAML, **Helm**, or **Kustomize**) and **reconciles** them onto the cluster.
- Features you use daily: **sync status (Synced/OutOfSync)**, **auto-sync**, **self-heal** (revert manual `kubectl` changes), **health checks**, **sync waves/hooks** for ordering, **rollback** to a previous Git revision, and **RBAC/SSO** for who can sync what.

### The GitOps pipeline (CI vs CD split)
```text
Dev push → CI (Jenkins/GitHub Actions): build, test, scan
        → docker build → push image to ECR (tag = git SHA / version)
        → CI updates the manifests repo: image tag 3.4 → 3.5  (commit/PR)
        ─────────────────────────────────────────────────────────────
Argo CD (CD): detects the Git commit → renders manifests →
              syncs to the cluster → Deployment rolls to 3.5
```
- **CI's job** ends at "push image + update the Git tag." **Argo CD's job** is everything after Git. This clean split is the point of GitOps: **Git is the single source of truth**, and the cluster is reconciled to match it.
- (In a Jenkins-based shop this is "GitOps-style": Jenkins builds and updates Git, Argo CD deploys. Not "pure" GitOps but the same model.)

### The crux: how does Argo CD know about `3.5` in ECR?
**By default, it doesn't** — it only reacts to **Git**. Three ways a new image reaches the cluster:
| Approach | How the new tag gets into Git |
|---|---|
| **CI updates the manifest** (most common) | The pipeline edits `image: repo:3.5` in the Git repo (commit or PR); Argo CD syncs the commit |
| **Argo CD Image Updater** | A component **polls ECR** for new tags matching a rule (semver/regex/newest-build) and **writes the new tag back to Git**, which Argo CD then syncs |
| **Manual** | A human bumps the tag in Git (fine for demos, not for scale) |

So the honest, correct answer to "how does Argo CD know there's a 3.5 in ECR?" is: **it knows because the image tag in Git changed** — pushed there by CI, or by **Argo CD Image Updater** polling the registry. Argo CD watching ECR directly is **not** the default behavior.

### Why "watch Git, not the registry" is deliberate
- **Auditability & rollback**: every deploy is a Git commit you can review and revert.
- **Reproducibility**: the cluster's desired state is fully described in Git.
- Auto-deploying whatever lands in a registry would bypass review and make rollbacks murky — hence the Git indirection.

### ⚠️ Things they probe
- "Does Argo CD poll ECR?" → **No, not by default; it reconciles Git.** New image = **new tag committed to Git** (CI) or **Argo CD Image Updater**.
- Know the **CI vs CD boundary**: CI builds/pushes/updates-Git; Argo CD deploys.
- Mention **auto-sync + self-heal** and **rollback = revert the Git commit**.

---

### 🧠 Real-world Example

> “CI (Jenkins) built the image, pushed it to **ECR** tagged with the git SHA, then ran a step that updated `image:` in our **Kustomize** overlay in the manifests repo. **Argo CD** (auto-sync + self-heal) picked up that commit and rolled the Deployment. Rollback was just reverting that commit. For a couple of fast-moving apps we used **Argo CD Image Updater** so new semver tags in ECR were written back to Git automatically.”

> Summary:  
> Argo CD is **GitOps**: it reconciles the cluster to **Git**, not to the registry. A new ECR image deploys only once its **tag is updated in Git** — by **CI** or **Argo CD Image Updater**. CI builds/pushes/updates-Git; Argo CD deploys, self-heals, and rolls back via Git.

### Related
- [App OutOfSync in Argo CD (drift)](./14-argocd-app-out-of-sync.md)
- [GitHub Actions OIDC → AWS (keyless CI)](../10-aws/26-github-actions-oidc-federation.md)
