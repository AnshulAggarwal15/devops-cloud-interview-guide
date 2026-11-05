## Question  
App in ‘OutOfSync’ State in Argo CD, But No Git Changes — What Could Be the Reason?

### 📝 Short Explanation  
In Argo CD, an application may show as **OutOfSync** even when Git hasn't changed, because the **live Kubernetes state differs from the desired Git state**.

## ✅ Answer  

This typically happens when **someone manually modified resources** in the cluster, or when **non-Git-managed changes** occur (e.g., automatic scaling or label updates).

---

### 🧭 Common Reasons and How to Fix Them

#### 1. 👨‍🔧 **Manual Changes in the Cluster**
Someone edited a deployment, config map, or secret directly using `kubectl edit`, `kubectl patch`, or via another tool (like Helm or the Kubernetes Dashboard).

✅ Fix:  
Revert manual changes by syncing the app via Argo CD:
```bash
argocd app sync <app-name>
```

---

#### 2. 🔄 **Dynamic Changes Not Tracked in Git**
Some fields (like annotations, replica counts via HPA) may change at runtime and cause drift.

✅ Options:
- Use `ignoreDifferences` in `Application` manifest to exclude these fields:
```yaml
spec:
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas
```

---

#### 3. 🪵 **Secrets Automatically Rotated**
If you're using tools like **Sealed Secrets**, **External Secrets**, or **Vault Agent Injector**, secrets might rotate or mutate at runtime.

✅ Fix:  
- Use `ignoreDifferences` for secret fields
- Or exclude secrets from Argo CD sync tracking

---

#### 4. ⏱️ **Argo CD Sync Window Missed**
If auto-sync is enabled but sync windows (time-based restrictions) are defined, changes might not be applied even if detected.

✅ Fix:
- Check for sync windows in Argo CD settings or annotations
- Trigger manual sync if needed

---

#### 5. 🔀 **CRDs or Hooks Trigger Drift**
If a Helm release contains post-install hooks or CRDs that modify resources post-sync, drift may be detected.

✅ Fix:
- Ensure generated resources are tracked properly
- Use Helm’s `skipHooks: true` if safe to ignore

---

### 🧠 Real-World Example

We had an application stuck in `OutOfSync` even though there were no Git changes.  
Root cause: A DevOps engineer had manually increased replica count on the Deployment to test scaling.

✅ Resolution:
- Re-synced the app from Argo CD to restore Git-desired state.
- Added `ignoreDifferences` to skip replica count drift in future.

---

> Summary:  
> Argo CD marks an app `OutOfSync` when the live state in the cluster doesn’t match Git — not just when Git changes.  
> Manual changes, runtime drift, or auto-generated mutations can cause this, and can be fixed with sync or exclusions.

---

### Related
- [Argo CD, GitOps & how a new image gets deployed](./16-argocd-gitops-and-image-updates.md) — what Argo CD does day to day, the CI→ECR→Git→sync pipeline, and why Argo CD watches **Git, not the registry** (Image Updater for auto image bumps).
