# 15 - Study Roadmaps

Personalized, gap-driven study plans built from **honest self-assessments** (spoken answers, no LLM). Each roadmap targets the specific weaknesses that assessment exposed and maps them to the notes elsewhere in this repo plus hands-on labs.

## The core principle
**Reading ≠ knowing.** This repo has ~50 solid notes; notes are a *reference*, not knowledge. Every roadmap uses the same loop:

> **Learn** (read the note) → **Do** (a hands-on lab, or at least say it) → **Prove** (close the note and say the mechanism out loud, in your own words). If you ramble or fall back to keywords, you haven't learned it — repeat tomorrow.

Target: be able to **teach each topic to an imaginary junior in 60 seconds**, unaided.

## Roadmaps
| Topic | Baseline | Roadmap |
|---|---|---|
| Kubernetes | ~4.5/10 (junior; disqualifying for mid-senior) | [01-kubernetes.md](./01-kubernetes.md) — two paths: 2–3 month mastery+CKA, and a limited-time crash path |
| Terraform | ~6.5/10 (solid mid-level) | [02-terraform-fixes-sts-oidc-and-modules.md](./02-terraform-fixes-sts-oidc-and-modules.md) — gap-fixes, the cross-domain **STS+OIDC** model (covers TF/K8s/CI-CD), and module structure |

*(More topics to come — AWS/networking, CI/CD — as each is assessed.)*

## Round-specific prep
| Doc | Use it for |
|---|---|
| [03-experience-db-scripting-fitment-round.md](./03-experience-db-scripting-fitment-round.md) | Experience-grill + DB + scripting + fitment round game plan |
| [04-deep-drill-round-prep.md](./04-deep-drill-round-prep.md) | Deep-drill prep: IDP, Dockerfile, secret rotation, EKS reliability, DB, coding + a **nerves** plan |
| [05-idp-talk-track.md](./05-idp-talk-track.md) | **IDP grill:** spoken STAR answers from real Jira work + the honest "adopter/extender, not core-builder" boundary |
| [06-database-from-scratch.md](./06-database-from-scratch.md) | **DB from zero:** one mental model + running example → JOINs, indexes, ACID, RDS bridge, slow-query playbook, Q&A |

> **Cross-domain:** the **STS + OIDC** section in `02-...` is the single highest-leverage concept — it fixes the identity/federation gap that showed up in *both* Kubernetes (IRSA) and Terraform (keyless CI auth).

## How to use
1. Read your baseline and the assessed gaps at the top of the roadmap.
2. Pick **Path 1** (real competence) unless an imminent interview forces **Path 2** (triage).
3. Work the phases in order; **don't skip the labs** — the whole lesson from the assessment is that reading didn't stick.
4. Re-drill earlier phases' one-liners weekly (spaced repetition) so they don't fade.

## Related
- Interview question banks & transcripts: [../14-previous-interview-questions](../14-previous-interview-questions)
