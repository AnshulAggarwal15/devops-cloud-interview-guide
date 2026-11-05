# Common AWS DevOps / Cloud / SRE Questions (3–5 yrs) — Coverage Map

A curated bank of the **standard** questions that recur across AWS DevOps / Cloud Engineer / SRE interviews (not from one company — the "everyone gets asked these" set). Use it as a **self-audit checklist**: each question links to its answer note.

**Legend:** ✅ covered in a dedicated note · ⭐ added while filling this bank · ⚠️ answered inline below (closest note linked)

> Takeaway from the source: at 3–5 years they test **real-time troubleshooting, production scenarios, CI/CD implementation, AWS architecture, K8s debugging, and incident handling** — mechanism and scenarios, not definitions.

---

## 🔹 Jenkins & CI/CD
| Question | Where |
|---|---|
| Design an end-to-end CI/CD pipeline | ⭐ [17 – CI/CD from scratch + GitOps](../05-cicd/17-cicd-from-scratch-gitops-deep-dive.md) |
| Store & manage credentials in Jenkins | ⭐ [18 – Freestyle vs Pipeline & credentials](../05-cicd/18-freestyle-vs-pipeline-and-jenkins-credentials.md) |
| Troubleshoot a failed Jenkins pipeline | ✅ [05 – build fails in CI not local](../05-cicd/05-build-fails-in-ci-but-not-local.md) · [debug-order playbook](../docusign-debug-order-playbook.md) |
| Freestyle vs Pipeline jobs | ⭐ [18](../05-cicd/18-freestyle-vs-pipeline-and-jenkins-credentials.md) |
| Approval before production deployment | ⭐ [17 – deployment gate](../05-cicd/17-cicd-from-scratch-gitops-deep-dive.md) |
| Integrate SonarQube & Trivy into Jenkins | ✅ [12 – static code analysis](../05-cicd/12-static-code-analysis-advantages.md) · Trivy in [17](../05-cicd/17-cicd-from-scratch-gitops-deep-dive.md) |

## 🔹 Git & GitHub
| Question | Where |
|---|---|
| `git merge` vs `git rebase` | ✅ [07](../02-git/07-git-merge-vs-rebase.md) / [08](../02-git/08-git-merge-vs-rebase-practical.md) / [09](../02-git/09-git-merge-vs-rebase-short-explanation.md) |
| Resolve merge conflicts | ✅ [13 – merge conflicts](../02-git/13-git-merge-conflicts.md) |
| Protect sensitive info from being pushed | ✅ [21 – remove secrets](../02-git/21-remove-secrets-from-git.md) · [18 – .gitignore](../02-git/18-gitignore.md) |
| Branching strategies | ✅ [10 – branching strategy](../02-git/10-git-branching-strategy.md) |

## 🔹 Docker
| Question | Where |
|---|---|
| Image vs container | ⭐ [13 – image vs container & Dockerfile optimization](../07-docker/13-image-vs-container-and-optimizing-dockerfiles.md) |
| Container that keeps restarting | ✅ [01 – exits immediately](../07-docker/01-docker-container-exits-immediately.md) (+ K8s [19 – CrashLoop](../08-kubernetes/19-crashloopbackoff.md)) |
| Optimize a Dockerfile | ⭐ [13](../07-docker/13-image-vs-container-and-optimizing-dockerfiles.md) |
| Push images to ECR | ✅ [09 – container registry](../07-docker/09-container-registry.md) · auth in ⭐ [17](../05-cicd/17-cicd-from-scratch-gitops-deep-dive.md) |

## 🔹 Kubernetes / EKS
| Question | Where |
|---|---|
| Troubleshoot CrashLoopBackOff | ✅ [19 – CrashLoopBackOff](../08-kubernetes/19-crashloopbackoff.md) |
| Troubleshoot ImagePullBackOff | ⭐ [43 – Pending/ImagePull/node-down](../08-kubernetes/43-troubleshooting-pending-imagepull-node-down.md) |
| Worker node goes down | ⭐ [43](../08-kubernetes/43-troubleshooting-pending-imagepull-node-down.md) |
| Deployment vs StatefulSet vs DaemonSet | ⭐ [42](../08-kubernetes/42-deployment-statefulset-daemonset.md) |
| Readiness & liveness probes | ✅ [20 – liveness vs readiness](../08-kubernetes/20-liveness-vs-readiness.md) |
| Zero-downtime deployment | ✅ [16 – deployment strategies](../08-kubernetes/16-all-deployment-strategies.md) |
| Pod stuck in Pending | ⭐ [43](../08-kubernetes/43-troubleshooting-pending-imagepull-node-down.md) |

## 🔹 AWS
| Question | Where |
|---|---|
| VPC, subnet, route table, security groups | ✅ [101 – VPC/subnets/routing](../10-aws/101-vpc-subnets-routing.md) · [102 – SG vs NACL](../10-aws/102-security-groups-vs-nacls.md) · [116 – VPC design](../10-aws/116-vpc-and-subnet-design.md) |
| ALB vs NLB vs CloudFront | ⚠️ [106 – load balancers](../10-aws/106-load-balancers-and-502.md) + CloudFront note below |
| Secure an AWS environment | ✅ [118 – cloud security fundamentals](../10-aws/118-cloud-security-fundamentals.md) |
| Troubleshoot an unreachable EC2 | ⚠️ inline below (+ [SSH stopped working](../03-linux/06-ssh-stopped-working.md), [connection refused](../03-linux/05-app-returns-connection-refused.md)) |
| Monitor apps using CloudWatch | ⚠️ inline below (+ [observability tools](../09-observability/07-observability-tools.md)) |
| Design a highly available app on AWS | ✅ [107 – HA & autoscaling](../10-aws/107-high-availability-and-autoscaling.md) · [01 – HA multi-tier](../10-aws/01-design-ha-scalable-multi-tier-app.md) |

## 🔹 SRE / Troubleshooting
| Question | Where |
|---|---|
| CPU suddenly 95–100% | ✅ [server slow / high CPU](../03-linux/04-server-slow-high-cpu-usage.md) |
| HTTP 503 errors | ⚠️ inline below (+ [502 bad gateway](../04-networking/06-502-bad-gateway.md)) |
| Latency 200 ms → 5 s | ✅ [slowness in app](../04-networking/04-slowness-in-app.md) · [slowness, no errors](../09-observability/08-slowness-no-errors-cpu-good.md) |
| Disk usage 95% | ✅ [/var full](../03-linux/03-var-folderi-is-full.md) · [remove old logs](../03-linux/08-remove-log-files-older-than-30-days.md) |
| Monitor availability & latency | ✅ [monitoring vs observability](../09-observability/01-monitoring-vs-observability.md) · [metrics to scrape](../09-observability/03-metrics-to-scrape-with-prometheus.md) |
| SLI, SLO, SLA | ⭐ [13 – SLI/SLO/SLA & error budgets](../09-observability/13-sli-slo-sla-error-budgets.md) |
| Handle a P1 production incident | ⭐ [14 – incident management & P1](../09-observability/14-incident-management-and-p1.md) |

---

## ⚠️ Inline answers for the partials (so nothing's unanswered)

**ALB vs NLB vs CloudFront**
- **ALB** (L7): HTTP/HTTPS, host/path routing, WAF, TLS termination — web apps/microservices.
- **NLB** (L4): TCP/UDP, ultra-low latency, static IP, extreme throughput — non-HTTP or performance-critical.
- **CloudFront** (CDN): caches content at **edge locations** globally, terminates TLS at the edge, integrates WAF, and fronts an origin (ALB/S3). Use it for **latency + offload + edge security**; it's not a load balancer, it's a cache/edge layer *in front of* one. (See [106](../10-aws/106-load-balancers-and-502.md).)

**Troubleshoot an unreachable EC2** (systematic path)
1. **Reachability:** is it running? status checks (system vs instance) passing?
2. **Network:** **security group** inbound (port 22/443 from your IP?), **NACL**, route table (IGW for public / correct routes), public IP/EIP present?
3. **Path:** correct subnet, is the ENI healthy, is it in a public subnet if you expect internet access?
4. **OS/app:** SSH daemon up? firewall (`ufw`/`iptables`)? disk full? Use **SSM Session Manager** or serial console if SSH is dead. (See [SSH stopped working](../03-linux/06-ssh-stopped-working.md).)

**Monitor apps using CloudWatch**
- **Metrics** (built-in + custom via PutMetricData/agent), **Logs** (CloudWatch Agent / Fluent Bit → log groups), **Alarms** (threshold/anomaly → SNS/Auto Scaling), **Dashboards**, **Container Insights** (EKS/ECS), and **Synthetics** canaries for availability. Tie alarms to **SLOs**, not raw CPU. (See [SLI/SLO/SLA](../09-observability/13-sli-slo-sla-error-budgets.md).)

**HTTP 503 vs 502** (they differ)
- **502 Bad Gateway** = the proxy/LB reached the backend but got an **invalid/failed response** (app crashed, wrong port, timeout). See [502 note](../04-networking/06-502-bad-gateway.md).
- **503 Service Unavailable** = **no healthy backend to send to** (all targets failing health checks, pods not `Ready`, or deliberate throttling/maintenance). On an ALB, 503 usually means the **target group has no healthy targets**.
- **Debug 503:** check target/endpoint health → are pods `Ready` (readiness probe)? → did a deploy remove all healthy targets? → capacity/throttling? Then scale/roll back to restore healthy targets.

---

## How to use this bank
1. Cover the right column and try to answer each question **out loud** in 60–90 seconds.
2. If you stumble, open the linked note and re-learn the **mechanism** (not just the definition).
3. Mark the ⭐ ones first — those were **gaps in this guide** until now, which means they're gaps a lot of candidates share; nailing them is easy differentiation.

> Related company transcripts: [Nielsen](./01-nielsen.md) · [Amagi](./02-amagi.md) · [Jio](./03-jio-cloud-security.md) · [MoveInSync](./04-moveinsync.md)
