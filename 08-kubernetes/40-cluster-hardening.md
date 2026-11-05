## Hardening a Kubernetes Cluster

### Question  
What does it mean to **harden** a Kubernetes cluster, and which **tools** do you use to do it?

### Short explanation of the question  
"Hardened" means the cluster is configured to reduce attack surface and blast radius — measured against a **benchmark** (usually the **CIS Kubernetes Benchmark**). The interviewer wants concrete controls across **control plane, nodes, workloads, network, and supply chain**, plus the **tools** that scan/enforce them (kube-bench, Kyverno/OPA, Trivy, Falco).

---

### Answer  
Hardening = applying least-privilege and defense-in-depth across layers and continuously verifying against the **CIS Benchmark**:
- **Control plane / API**: no anonymous auth, RBAC least privilege, audit logging, encrypt etcd at rest.
- **Workloads**: **Pod Security Standards (restricted)**, drop root/privileged, read-only rootfs, admission policies (**Kyverno/OPA Gatekeeper**).
- **Network**: default-deny **NetworkPolicies**, restrict pod access to **IMDS**.
- **Supply chain**: scan images (**Trivy**), signed images, trusted registries.
- **Runtime**: threat detection (**Falco**).
- **Tools**: **kube-bench** (CIS checks), **kube-hunter** (pen-test), **Trivy** (images/IaC), **Kyverno/Gatekeeper** (policy), **Falco** (runtime).

---

### Detailed explanation of the answer for readers’ understanding

### 🛡️ What "hardened" covers (by layer)
| Layer | Key controls |
|---|---|
| **API server / control plane** | Disable **anonymous auth**, strong authn (OIDC), **RBAC** least privilege, **audit logs**, **etcd encryption at rest**, restrict/secure kubelet API |
| **Nodes** | Minimal OS, patched, no SSH sprawl, restrict **IMDS** (hop limit / block from pods), read-only where possible |
| **Workloads / pods** | **Pod Security Standards `restricted`**, `runAsNonRoot`, drop Linux capabilities, `readOnlyRootFilesystem`, no `privileged`, no `hostPath`/`hostNetwork` unless required, seccomp/AppArmor |
| **Network** | **Default-deny NetworkPolicies**, segment namespaces, control **egress**, mTLS via service mesh if needed |
| **Secrets** | Encrypt at rest (KMS), external secret store, tight RBAC on `secrets` |
| **Supply chain** | **Image scanning** (Trivy), signed images (Cosign), trusted registries, no `latest` tag |
| **Runtime** | **Falco** for anomalous syscalls/behavior; drift/threat detection |

### 🧰 The tools (name these)
| Tool | Purpose |
|---|---|
| **kube-bench** | Automated **CIS Kubernetes Benchmark** checks against your cluster |
| **kube-hunter** | Actively probes the cluster for exploitable weaknesses |
| **Trivy** | Scans **container images** and **IaC/manifests** for CVEs and misconfig |
| **Kyverno / OPA Gatekeeper** | **Admission policies** that *block* non-compliant workloads (e.g. deny privileged pods) |
| **Pod Security Admission** | Built-in enforcement of Pod Security Standards (`baseline`/`restricted`) |
| **Falco** | **Runtime** threat detection (unexpected shells, file/network activity) |

### 🚫 Prevention vs enforcement (the "make them actually block" nuance)
- **Scanning** (kube-bench, Trivy) *reports* problems — valuable, but advisory.
- To **enforce**, wire the scan as a **blocking CI gate** (fail the pipeline on high-severity findings) and use **admission controllers** (Kyverno/Gatekeeper) so a non-compliant pod is **rejected at deploy time**, not just flagged. This is exactly what interviewers mean by "made them actually block."

### 🧭 A practical hardening baseline (say this)
1. Run **kube-bench**, fix the CIS failures (anonymous auth, RBAC, audit, etcd encryption).
2. Enforce **Pod Security `restricted`** + **Kyverno** policies (no privileged/root/hostPath).
3. Apply **default-deny NetworkPolicies**, then allow only needed flows; restrict IMDS.
4. Gate images with **Trivy** in CI (block criticals) + use **IRSA** for pod AWS access (no node-role reliance).
5. Add **Falco** for runtime detection and ship findings to your SIEM.

### ⚠️ Things they probe
- Anchor hardening to the **CIS Benchmark** and name **kube-bench**.
- Distinguish **scan/report** vs **admission/block** — enforcement is the point.
- Least privilege everywhere: **RBAC**, **PSS restricted**, **default-deny network**, **IRSA over node role**.

---

### 🧪 Real-World Example

> “We baselined with **kube-bench** and fixed the CIS gaps (disabled anonymous auth, tightened RBAC, enabled audit + etcd encryption). **Kyverno** admission policies rejected privileged/root pods, **NetworkPolicies** were default-deny, and **Trivy** blocked images with critical CVEs in CI. **Falco** watched runtime behavior. Pods used **IRSA**, and we raised the IMDS hop limit story into 'give pods their own role' rather than the node role.”

---

### Key takeaway

> “Hardening = **least privilege + defense in depth vs the CIS Benchmark** across control plane, nodes, workloads, network, and supply chain. Verify with **kube-bench/kube-hunter/Trivy**, and **enforce** with **admission policies (Kyverno/OPA) + blocking CI gates** so non-compliant workloads are rejected, not just reported. Add **Falco** for runtime.”

### Related
- [Kubernetes RBAC](./36-rbac.md) · [Network policies](./12-network-policies.md)
- [Cloud security fundamentals](../10-aws/118-cloud-security-fundamentals.md)
- [Pod IAM / IMDS troubleshooting](../10-aws/27-pod-node-role-imds-troubleshooting.md)
