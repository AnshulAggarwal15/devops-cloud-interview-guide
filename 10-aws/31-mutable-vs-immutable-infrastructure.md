<!-- #genai: interview note - mutable vs immutable infrastructure; patch-in-place vs replace, AMIs/containers/IaC, benefits, blue-green tie-in -->

## Mutable vs Immutable Infrastructure

### Question
What's the difference between **mutable** and **immutable** infrastructure, and why does immutable win for modern deployments?

### Short Explanation
**Mutable** = you **change servers in place** (SSH in, patch, update, re-run config management). **Immutable** = you **never modify a running server** — to change anything you **build a new image/instance and replace** the old one. Immutable is the modern default because it kills **configuration drift** and makes deploys **predictable and easily reversible**.

## Answer
- **Mutable infrastructure:** long-lived servers updated in place (apt/yum upgrades, Ansible runs, hotfixes). Over time each server accumulates unique changes → **drift** ("snowflake servers").
- **Immutable infrastructure:** servers are **never changed after creation**. A new version = a **new artifact** (AMI/container image) rolled out to **fresh instances**, and the old ones are destroyed. No in-place edits.
- **Why immutable wins:** **no drift**, **reproducible/identical** environments, **trivial rollback** (redeploy the previous image), and it fits **containers, autoscaling, and blue-green/canary** naturally.

## Detailed Explanation

### Side by side
| | Mutable | Immutable |
|---|---|---|
| Change model | **Patch in place** (SSH/config mgmt) | **Replace** with a new build |
| Drift | Accumulates → snowflakes | **None** (rebuilt each time) |
| Rollback | Reverse the change (fragile) | **Redeploy previous image** (clean) |
| Reproducibility | Hard ("works on that box") | **Identical every time** |
| Fits | Legacy/pets | **Containers, ASG, blue-green/canary** |
| Tooling | Ansible/Chef/Puppet in-place | **Packer AMIs, Docker images, IaC replace** |

### What "immutable" looks like in practice
- **Containers** are the purest example: you don't `exec` in to patch a running container — you **build a new image tag** and redeploy. (Ties to immutable image tags = git SHA/digest, never `latest`.)
- **AMIs (Packer):** bake a golden AMI per release; an ASG **replaces instances** with the new AMI rather than patching live ones.
- **IaC replace semantics:** changing an immutable attribute makes Terraform **destroy + recreate** (and `create_before_destroy` avoids downtime). You ship change as **new infra**, not edits.
- **Config as code:** all configuration is in the image/manifest, not applied ad-hoc to a live box.

### Why it's the modern default (say this)
- **No configuration drift** — every instance came from the same artifact, so "it works here but not there" disappears.
- **Predictable deploys & easy rollback** — deploy = swap to a new image; rollback = swap back to the previous one (this is exactly how **blue-green/canary** and GitOps rollback work).
- **Scales cleanly** — autoscaling launches identical instances from one image; no bootstrap-time patching races.
- **Better security posture** — short-lived servers, patched by rebuild (rebuild the base image + redeploy) rather than long-lived boxes drifting out of compliance.

### The trade-offs (be honest)
- You need a **solid build/artifact pipeline** (Packer/Docker + registry) and **externalized state** — immutable servers must be **stateless**, so data lives in RDS/S3/EFS/volumes, not on the box.
- Rebuild-and-replace can feel heavier than a quick patch — but that "quick patch" is exactly what creates drift.

### ⚠️ Things they probe
- **Immutable = replace, never modify in place**; mutable = patch in place → **drift**.
- Containers and **golden AMIs (Packer)** are the canonical examples; **rollback = redeploy the old image**.
- Immutable requires **stateless** compute (externalize data) and a **build pipeline**.
- Connect it to **blue-green/canary** and **GitOps** — same "ship a new version, keep the old to roll back" idea.

## Summary
- **Mutable:** change servers in place → drift, fragile rollbacks, snowflakes.
- **Immutable:** build a **new image/instance and replace** → no drift, reproducible, clean rollback, scales well.
- Powered by **containers / Packer AMIs / IaC replace**, requires **stateless** compute + a build pipeline, and underpins **blue-green, canary, and GitOps rollback**.

## Related
- [ASG & scaling](./115-asg-and-scaling.md) · [EKS migration (blue/green)](./120-eks-migration.md)
- [Image vs container & Dockerfile optimization](../07-docker/13-image-vs-container-and-optimizing-dockerfiles.md)
- [Deployment strategies (blue-green/canary)](../08-kubernetes/16-all-deployment-strategies.md) · [CI/CD from scratch + GitOps](../05-cicd/17-cicd-from-scratch-gitops-deep-dive.md)
