## Developer Deleted Critical Resources (S3, RDS, EC2) — What Will You Do?

### Question  
A developer accidentally deleted critical resources like S3, RDS, and EC2. What will you do?

### Short explanation of the question  
This is both a **recovery** question and a **prevention** question. The interviewer wants to see calm incident handling (recover from backups/snapshots), then the guardrails that stop it happening again (least privilege, deletion protection, versioning, backups).

### Answer  
First I **recover** each resource from its backup mechanism — S3 from **versioning/replication**, RDS from **snapshots or point-in-time restore**, EC2 from **AMIs/EBS snapshots**. Then I **prevent recurrence** with least-privilege IAM, deletion protection, S3 versioning + MFA delete, and reliable backups.

### Detailed explanation of the answer

#### 1. 🧯 Recover first (per service)

| Resource | Recovery path |
|---|---|
| **S3 object** | If **versioning** is on, delete the *delete marker* to restore. Cross-region replication as backup. |
| **S3 bucket** | Recreate; restore objects from replication/backup. Bucket names are global, so recreate carefully. |
| **RDS** | Restore from an **automated snapshot** or **point-in-time recovery** (within the retention window). |
| **EC2** | Relaunch from the latest **AMI**; reattach data from **EBS snapshots**. |

#### 2. 🔒 Then prevent it from happening again

- **Least-privilege IAM:** developers shouldn't hold `Delete*` on production. Separate prod/dev accounts (AWS Organizations).
- **Deletion / termination protection:** enable **RDS deletion protection**, **EC2 termination protection**, and S3 **MFA Delete**.
- **S3 versioning** so deletes are recoverable, plus lifecycle rules.
- **Backups:** **AWS Backup** plans, tested restores, cross-region copies.
- **Guardrails:** **SCPs** to deny deletion of tagged critical resources; require change approval.
- **CloudTrail** to audit who did what.

#### 3. 🔁 Do a blameless postmortem
Focus on the missing guardrail, not the person.

---

### 🧠 Real-world Example

> A dev deleted a production RDS thinking it was a test DB. We restored via **point-in-time recovery** to a few minutes before deletion. Afterwards we enabled **deletion protection**, split prod into its own account, and applied an **SCP** denying `rds:DeleteDBInstance` on prod-tagged databases.

### Key takeaway

> "**Recover from backups** (S3 versioning, RDS snapshots/PITR, EC2 AMIs/EBS snapshots), then **close the gap** with least-privilege IAM, deletion protection, versioning, and tested backups. Accidental deletion should always be recoverable — and hard to do in the first place."
