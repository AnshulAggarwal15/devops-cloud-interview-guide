## Getting Pod Logs With No Observability Stack

### Question  
No observability stack, no Argo CD access — the pod is running (and had an earlier crash). **How do you get its logs?** Once you `exec` inside the pod, can you see the pod's logs there? And where do "pod logs" actually come from?

### Short explanation of the question  
This checks that you can debug with **plain `kubectl`** and understand the **logging mechanism**: the app writes to **stdout/stderr**, the **container runtime** captures those streams, the **kubelet** exposes them, and **`kubectl logs`** fetches them via the **Kubernetes API**. The trap: **`kubectl logs` ≠ going inside the container** — if the app only logs to stdout, there's no log file inside to read, and you usually can't run `kubectl` from inside the pod.

---

### Answer  
- Get logs directly with **`kubectl logs`** — no observability tool needed:
```bash
kubectl logs <pod> -n <ns>                     # current container stdout/stderr
kubectl logs <pod> -c <container> -n <ns>      # a specific container (multi-container pod)
kubectl logs -f <pod> -n <ns>                  # stream live
kubectl logs <pod> -n <ns> --previous          # logs from the PREVIOUS (crashed/OOMKilled) container
```
- **`kubectl logs`** returns the container's **stdout/stderr** (captured by the runtime, served via the API). **`kubectl exec`** puts you *inside* the container — different thing.
- **Inside the pod you generally cannot get the Kubernetes container logs**: if the app logs to stdout there's no file, and `kubectl` isn't installed/authorized inside an app container. Inside, you can only read **application log files** *if* the app writes them.

---

### Detailed explanation of the answer for readers’ understanding

### 📜 Where "pod logs" come from (the mechanism they probe)
```text
App writes to stdout / stderr
        │
Container runtime (containerd) captures the streams
        │  writes to the node: /var/log/pods/... and /var/log/containers/*.log
kubelet exposes them
        │
kubectl logs  ──(Kubernetes API)──►  returns the container logs
```
- So the app **doesn't push logs anywhere special** — it just writes to **stdout/stderr**, and Kubernetes (runtime + kubelet) captures and exposes them. `kubectl logs` reads that through the **API server**.
- On the **node**, those logs live at **`/var/log/pods/<ns>_<pod>_<uid>/<container>/*.log`** and symlinked under `/var/log/containers/`. (Relevant if you have node/SSH access but not API access.)

### 🔍 `kubectl logs` vs `kubectl exec` (the key distinction)
| | `kubectl logs` | `kubectl exec` |
|---|---|---|
| Gets you | The container's **stdout/stderr** (via API) | A **shell inside** the running container |
| Needs the app to… | log to **stdout/stderr** (the K8s convention) | — |
| Sees app **log files**? | No | **Only if** the app writes files inside |
| Works after a crash? | **Yes, `--previous`** | No (need a running container) |

```bash
kubectl exec -it <pod> -n <ns> -- /bin/sh
# then, inside, look for FILE logs the app may have written:
ls -lah /var/log
find / -name "*.log" 2>/dev/null
tail -f /app/logs/app.log
```

### ❓ "Can you see the pod's logs from inside via exec?"
**No — not the Kubernetes container logs.** Two reasons:
1. If the app logs to **stdout/stderr** (the standard), there's **no log file inside** the container to read — those streams were captured by the runtime, not written to disk in the container.
2. `kubectl` is a **client that talks to the API server**; it's normally **not installed or credentialed inside** an application container, so you can't run `kubectl logs` from in there.
So for container logs you **exit and run `kubectl logs` from a machine with API access**. Use `exec` only to inspect **files/processes/memory** the app itself created.

### 🩺 Practical debug flow (no observability)
1. `kubectl get pods -n <ns>` — find the pod / see restarts.
2. `kubectl describe pod <pod>` — Events (scheduling, pull, probe, OOM).
3. `kubectl logs <pod> --previous` — the crash logs (most valuable after a restart).
4. `kubectl logs -f <pod>` — watch it live.
5. `kubectl exec -it <pod> -- sh` — only if you need to inspect files/env/processes inside.

### ⚠️ Things they probe
- **App → stdout/stderr → runtime → kubelet → `kubectl logs` via API.** Say this chain.
- **`--previous`** for a crashed/OOMKilled container's logs.
- **`kubectl logs` (stdout) ≠ `exec` (inside).** If the app logs to stdout, `exec` won't show those logs.
- Node path **`/var/log/pods` / `/var/log/containers`** if you only have node access.
- Best practice: apps should **log to stdout/stderr** (12-factor) so the platform can collect them.

---

### 🧪 Real-World Example

> "A pod had restarted overnight with no logging stack in place. I ran `kubectl describe` (saw an OOMKill event), then `kubectl logs <pod> --previous` to get the stack trace from the killed container — that was the actual clue. I only `exec`-ed in afterwards to check a config file. I didn't need Argo CD or ELK; `kubectl logs` reads stdout/stderr straight from the API."

---

### Key takeaway

> "Use **`kubectl logs`** (with **`--previous`** for the crashed container, `-c` for a specific container, `-f` to stream) — it returns the container's **stdout/stderr**, captured by the runtime/kubelet and served via the API, **no observability stack required**. **`exec`** only gets you *inside* to read files the app wrote; if the app logs to stdout there's nothing to read inside, and you can't run `kubectl` from within the pod — exit and use `kubectl logs`."

### Related
- [CrashLoopBackOff](./19-crashloopbackoff.md) · [Pending/ImagePull/node-down troubleshooting](./43-troubleshooting-pending-imagepull-node-down.md)
- [Logs, metrics, traces](../09-observability/05-logs-metrics-traces.md) · [Debug-order playbook](../docusign-debug-order-playbook.md)
