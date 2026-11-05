## Kubernetes RBAC (Role-Based Access Control)

### Question  
Explain RBAC in Kubernetes — Roles vs ClusterRoles, bindings, and how it ties into EKS/IAM.

### Short explanation of the question  
RBAC is **how Kubernetes decides who can do what** on the API. The interviewer wants the four core objects (**Role, ClusterRole, RoleBinding, ClusterRoleBinding**), the difference between **namespaced vs cluster-wide**, and — for EKS — how **IAM identities map to Kubernetes RBAC**.

---

### Answer  
- **RBAC** grants permissions to **subjects** (users, groups, ServiceAccounts) through **roles** that list allowed **verbs on resources**.
- **Role** = permissions **within one namespace**; **ClusterRole** = cluster-wide (or reusable across namespaces, plus cluster-scoped resources like nodes).
- A **RoleBinding**/**ClusterRoleBinding** ties a role to subjects. RBAC is **purely additive** (no deny) — access is denied unless a rule allows it.

---

### Detailed explanation of the answer for readers’ understanding

### 🧩 The four objects
| Object | Scope | Purpose |
|---|---|---|
| **Role** | Namespaced | Verbs on resources **in one namespace** |
| **ClusterRole** | Cluster | Cluster-scoped resources (nodes, PVs) **or** a reusable role |
| **RoleBinding** | Namespaced | Bind a Role **or** a ClusterRole to subjects **in a namespace** |
| **ClusterRoleBinding** | Cluster | Bind a ClusterRole to subjects **cluster-wide** |

> Tip: a **RoleBinding can reference a ClusterRole** — define the permission set once, grant it namespace-by-namespace.

### 📄 Example — read-only pods in one namespace
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: dev
subjects:
- kind: ServiceAccount
  name: my-app
  namespace: dev
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### 👤 Subjects
- **ServiceAccount** — identity for **pods/workloads** (most common in automation).
- **User / Group** — humans or external identities; Kubernetes doesn't store users, it **trusts the authenticator** (certs, OIDC, or the cloud provider).

### ☁️ RBAC on EKS (IAM → Kubernetes)
- IAM handles **authentication**; Kubernetes RBAC handles **authorization**.
- The mapping lives in **EKS access entries** (newer) or the legacy **`aws-auth` ConfigMap**: an IAM role/user maps to a Kubernetes **user/group**, and that group is bound to RBAC roles.
- Pods get AWS permissions separately via **IRSA / EKS Pod Identity** — that's IAM for the pod, distinct from in-cluster RBAC.

### ✅ Best practices
- **Least privilege**: prefer **Role + RoleBinding** (namespaced) over cluster-wide bindings.
- One **ServiceAccount per workload**; never rely on the `default` SA with broad rights.
- Avoid binding to the built-in **`cluster-admin`** except for break-glass.
- Audit with `kubectl auth can-i --list --as=...` and reviews.

### 🔎 Verify permissions
```bash
kubectl auth can-i list pods --namespace dev --as system:serviceaccount:dev:my-app
kubectl auth can-i --list --as system:serviceaccount:dev:my-app -n dev
```

---

### 🧪 Real-World Example

> “Each microservice got its **own ServiceAccount** bound to a namespaced **Role** with just the verbs it needed. CI used a dedicated role mapped through **EKS access entries** (IAM → group → RBAC), and only the platform team's IAM role mapped to `cluster-admin` for break-glass. `kubectl auth can-i` was our quick audit.”

---

### Key takeaway

> “RBAC = **subjects → roles → verbs on resources**. **Role/RoleBinding = namespaced, ClusterRole/ClusterRoleBinding = cluster-wide**, and RBAC is **additive with no deny**. On EKS, **IAM authenticates**, then **access entries/`aws-auth` map the identity to a group** that RBAC authorizes — while pods get AWS access via **IRSA**.”

### Related
- [AWS IAM users, roles & policies](../10-aws/103-iam-users-roles-policies.md)
- [IRSA — how an EKS pod gets AWS credentials](../10-aws/104-irsa.md)
- [Types of secrets](./35-types-of-secrets.md)
- [Authentication vs Authorization framework](../10-aws/29-authentication-vs-authorization.md) — RBAC is the **authZ** half; IAM/OIDC is the **authN** half
