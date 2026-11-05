# Platform engineer job preparation
_Exported on 25/05/2026 at 20:04:32 GMT+5:30 from Cursor (2.5.25)_

---

**User**

Platform Engineer
Bangalore / Full-Time
Job Description
At Mareana, we build Enterprise-Scale AI/ML-powered products for Manufacturing,
Sustainability, and Supply Chain. As our platform continues to scale across multiple cloud
environments and customer deployments, we are strengthening our Platform Engineering
capabilities.
We are looking for a Platform Engineer who is passionate about building automation,

simplifying infrastructure operations, and enabling developers to move faster through self-
service platform capabilities. The role focuses on developing internal platform tooling,

implementing GitOps-driven deployments, and managing scalable cloud infrastructure across
AWS and Azure environments.
You will work closely with DevOps, engineering, and architecture teams to build standardized
deployment pipelines, reusable infrastructure components, and automation that reduces
operational complexity.
What You’ll Do
• Design and evolve Mareana’s Internal Developer Platform (IDP) to enable developers
to deploy infrastructure and applications through self-service workflows.
• Identify repetitive manual processes across development and operations and
implement automation to improve reliability and efficiency.
• Implement and maintain GitOps-driven deployment workflows using tools such as
FluxCD or ArgoCD for Kubernetes environments.
• Develop and manage CI/CD pipelines using Jenkins and Git-based workflows to
automate build, testing, and release processes.
• Provision and manage scalable cloud infrastructure across AWS and Azure using
automation and infrastructure-as-code practices.
• Build reusable infrastructure modules, deployment templates, and internal tooling to
standardize environment provisioning.
• Develop automation scripts and platform utilities using Python to improve system
reliability and reduce operational overhead.
• Implement monitoring, logging, and observability integrations to improve platform
visibility and operational response.

Requirements & Skills
• Bachelor’s degree in Computer Science, Information Technology, or a related field.
• 4–6 years of hands-on experience in Cloud Engineering, or Platform Engineering
roles.
• Experience working with cloud platforms including AWS and Azure.
• Experience with CI/CD tools such as Jenkins, GitLab CI/CD, or similar tools.
• Familiarity with GitOps workflows and tools such as FluxCD or ArgoCD.
• Experience with Kubernetes and containerized applications.
• Exposure to Kubernetes platform extension concepts such as Custom Resource
Definitions (CRDs) and development of Kubernetes Custom Operators.
• Strong scripting and automation skills using Python and Golang.
• Understanding of Git-based workflows and modern DevOps practices.
• Knowledge of monitoring and observability tools such as Prometheus, Grafana, or
similar platforms.
About Mareana
Founded in 2015, Mareana is an AI-powered software company with the mission of
accelerating digital transformation in manufacturing, supply chain, and sustainability via our
connected intelligence platform.
Mareana’s platform uses AI/ML to rapidly connect disparate, siloed data across the entire
business process, allowing our customers to shift their time and effort from data preparation to
making complex business decisions intuitively, in real time.
Our customers are market leaders in life sciences, chemicals, and general manufacturing who
have realized over a billion dollars in business value by leveraging our platform.
Our ethos of continuous innovation has been recognized by Gartner, who named us a “Cool
Vendor in AI”.

How to prepare for this?

---

**Cursor**

I see you've built up a solid prep library already (Linux, AWS, K8s, GitOps with ArgoCD, Terraform, Docker, CI/CD, Observability, Python/Bash, Behavioral). The Mareana JD has a few specific emphases your existing notes may not fully cover — namely **Internal Developer Platforms (IDP)**, **FluxCD** (you have ArgoCD), **Azure** (you have AWS), **Kubernetes CRDs/Custom Operators**, and **Golang**.

Before I draft a focused prep plan, two quick clarifications so I tailor it correctly.

Got it — 1 day, 1st technical round, IDP-focused with theory + situational. You already have strong notes for Linux/AWS/K8s/ArgoCD/Docker/CI-CD/Observability, so the plan below focuses **only on the Mareana-specific gaps** and gives you a tight 1-day schedule.

---

**User**

Implement the plan as specified, it is attached for your reference. Do NOT edit the plan file itself.

To-do's from the plan have already been created. Do not create them again. Mark them as in_progress as you work, starting with the first one. Don't stop until you have completed all the to-dos.

---

**Cursor**

I'll generate comprehensive study notes for each block as actual files in your Learn folder so you can cram from them today. Each file will be self-contained and interview-ready.

All 8 prep blocks are now in your `/Users/anshag-blrm24/Desktop/Learn/` folder, each as a self-contained, interview-ready study file:

| File | Block | Use it for |
|------|-------|-----------|
| [16-idp-fundamentals.md](Desktop/Learn/16-idp-fundamentals.md) | 1 — IDP | The #1 topic. CNCF 5-plane model, golden paths, Backstage, Team Topologies, 60-sec elevator pitch, likely Q&A. |
| [17-fluxcd-vs-argocd.md](Desktop/Learn/17-fluxcd-vs-argocd.md) | 2 — GitOps | Flux's GitOps Toolkit (5 controllers), side-by-side table, image-automation difference, 5-question decision framework. |
| [18-crds-and-operators.md](Desktop/Learn/18-crds-and-operators.md) | 3 — CRDs/Operators | Reconcile loop, Kubebuilder vs Operator SDK, capability levels, real-world operators to name-drop, Mareana `TenantEnvironment` angle. |
| [19-azure-quick-map.md](Desktop/Learn/19-azure-quick-map.md) | 4 — Azure | AWS↔Azure service table, Workload Identity = IRSA equivalent, AKS specifics, multi-cloud rules. |
| [20-golang-awareness.md](Desktop/Learn/20-golang-awareness.md) | 5 — Go | Why Go dominates K8s, syntax to recognize, `client-go` snippets, Python↔Go map, the honest "ramp in 60 days" pitch. |
| [21-mareana-star-stories.md](Desktop/Learn/21-mareana-star-stories.md) | 6 — STAR | 5 templated stories tuned for platform-as-product framing — fill the `[BRACKETS]` with your real history. |
| [22-mareana-context.md](Desktop/Learn/22-mareana-context.md) | 7 — Mareana | Why-the-platform-is-hard analysis, "Why Mareana" 3-thread answer, ranked questions to ask, traps to avoid. |
| [23-mareana-cheat-sheet.md](Desktop/Learn/23-mareana-cheat-sheet.md) | 8 — Cheat sheet | One-page glance file. Open this in a second window during the call. |

### Suggested order for the 1 day

1. **Now → +1.5h**: Read [16](Desktop/Learn/16-idp-fundamentals.md) twice. Memorize the 4 phrases and the 5-plane model.
2. **+1.5h → +2.5h**: [17](Desktop/Learn/17-fluxcd-vs-argocd.md) once + [18](Desktop/Learn/18-crds-and-operators.md) once.
3. **+2.5h → +3.5h**: [19](Desktop/Learn/19-azure-quick-map.md) + [20](Desktop/Learn/20-golang-awareness.md).
4. **+3.5h → +5h**: [21](Desktop/Learn/21-mareana-star-stories.md) — **fill the `[BRACKETS]` with real details from your history**, then read each story aloud with a timer.
5. **+5h → +5.5h**: [22](Desktop/Learn/22-mareana-context.md) — read once, pick your 3-4 questions.
6. **+5.5h → +6.5h**: Mock interview aloud using section N of [23](Desktop/Learn/23-mareana-cheat-sheet.md).
7. **Pre-sleep**: Skim [23](Desktop/Learn/23-mareana-cheat-sheet.md) once. Close laptop. Sleep early.

Two important reminders the cheat sheet will repeat tomorrow morning:
- The **most likely topic** is IDP — practice that elevator pitch until it's automatic.
- For FluxCD vs ArgoCD, **never commit to a tool without stating criteria first** — that single habit will make you sound senior.

Good luck.

