## Helm Essentials — Charts, Values, `range`, and Passing Values

### Question  
What is Helm? What's the difference between **values and parameters**, what does the **`range`** function do, and how do you **pass values** during a deployment?

### Short explanation of the question  
Helm is the **Kubernetes package manager**. Interviewers check that you understand a **chart = templated manifests + a `values.yaml`**, how to **override values** at deploy time (`-f` / `--set`), and basic templating like **`range`** (loop) — because that's what you touch daily when you deploy with Helm.

---

### Answer  
- **Helm** packages Kubernetes manifests into a reusable **chart**: templates (`templates/*.yaml`) + default config (`values.yaml`) + metadata (`Chart.yaml`). Install a chart → a **release**.
- **Values** are the configurable inputs defined in `values.yaml` and referenced in templates as `.Values.x`. "**Parameters**" usually means the specific values you **pass/override at deploy time** — there's no separate Helm object; passing a parameter = overriding a value.
- **`range`** is Helm's **loop** (Go templates): iterate over a list/map in `values.yaml` to generate repeated YAML (env vars, containers, ingress hosts).
- **Pass values** with **`-f myvalues.yaml`** (a file, precedence-ordered) and/or **`--set key=value`** (inline override). `--set` wins over `-f`, which wins over `values.yaml` defaults.

---

### Detailed explanation of the answer for readers’ understanding

### 📦 What a chart is
```text
mychart/
├── Chart.yaml        # name, version, appVersion
├── values.yaml       # default configuration (the inputs)
├── templates/        # templated manifests using {{ .Values.* }}
│   ├── deployment.yaml
│   ├── service.yaml
│   └── _helpers.tpl  # reusable template snippets
└── charts/           # dependency (sub)charts
```
- `helm install myrelease ./mychart` renders the templates with the values and applies them → a **release** (Helm tracks its revision history for rollback).
- Same chart, **different `values.yaml` per env** → deploy to dev/stage/prod without copy-pasting YAML.

### 🎛️ Values vs "parameters"
- **`values.yaml`** holds the **defaults**; templates read them as `.Values.replicaCount`, `.Values.image.tag`, etc.
- A **"parameter"** in interview-speak is just a **value you override** at install/upgrade time. Precedence (lowest → highest):
```text
chart values.yaml  <  -f custom-values.yaml  <  --set key=value
```
Example template use:
```yaml
spec:
  replicas: {{ .Values.replicaCount }}
  containers:
    - image: "{{ .Values.image.repo }}:{{ .Values.image.tag }}"
```

### 🔁 The `range` function (looping)
`range` iterates over a list or map from values to emit repeated blocks. Given:
```yaml
# values.yaml
env:
  - name: LOG_LEVEL
    value: info
  - name: REGION
    value: ap-south-1
```
```yaml
# templates/deployment.yaml
env:
{{- range .Values.env }}
  - name: {{ .name }}
    value: {{ .value | quote }}
{{- end }}
```
Inside `range`, the **`.` (dot) is rebound to the current item**; use `$` to reach the root scope (e.g. `$.Values.global`). `range` also works over maps: `{{- range $key, $val := .Values.labels }}`.

### 🚀 Passing values at deploy time
```bash
# a values file (recommended for anything non-trivial)
helm upgrade --install myapp ./mychart -f values-prod.yaml

# inline overrides (great in CI to set the image tag)
helm upgrade --install myapp ./mychart --set image.tag=abc123

# multiple -f files: later files override earlier ones
helm upgrade --install myapp ./mychart -f base.yaml -f prod.yaml
```
- In **CI/CD**, the pipeline typically `--set image.tag=<git-sha>`; in **GitOps**, the values file lives in Git and Argo CD renders the chart.

### 🧩 Other bits worth naming
- **`helm template`** renders locally (no cluster) to debug output; **`helm lint`** validates; **`helm rollback`** reverts a release revision.
- **`_helpers.tpl`** + `define`/`include` for reusable snippets; `if/else`, `with`, `default`, `required` for logic.
- **Helm vs Kustomize**: Helm = templating + packaging + releases; Kustomize = template-free overlays. Both common; often Argo CD renders one of them.

### ⚠️ Things they probe
- **Chart = templates + `values.yaml`**; install → **release** (with revision history for rollback).
- **`--set` > `-f` > `values.yaml`** precedence.
- **`range`** loops and rebinds `.`; use `$` for root scope.
- Override the **image tag** at deploy via `--set` (CI) or a values file (GitOps).

---

### 🧪 Real-World Example

> "Our chart had one `values.yaml` of defaults and per-env files (`values-prod.yaml`). CI ran `helm upgrade --install --set image.tag=$GIT_SHA`, and env-specific config (replicas, resources, ingress hosts) came from the `-f` file. We used `range` to template a variable list of env vars and ingress hosts, and `helm rollback` when a release went bad."

---

### Key takeaway

> "**Helm = K8s package manager**; a **chart = templates + `values.yaml`**, installed as a **release**. **Values** are the inputs (`.Values.*`); a **parameter** is just a value you override — precedence **`--set` > `-f` > `values.yaml`**. **`range`** loops over lists/maps (rebinding `.`, use `$` for root). Pass values with **`-f`** files and **`--set`** overrides (CI sets the image tag)."

### Related
- [Argo CD, GitOps & image updates](../05-cicd/16-argocd-gitops-and-image-updates.md) · [CI/CD from scratch + GitOps](../05-cicd/17-cicd-from-scratch-gitops-deep-dive.md)
- [App → Kubernetes production readiness](./41-app-to-kubernetes-production-readiness.md)
