## Question  
Explain your day-to-day activities related to databases.

### 📝 Short Explanation  
An experience question. The interviewer wants to confirm you actually **operate** databases (mostly **RDS/DynamoDB** as a DevOps engineer) — backups, monitoring, scaling, access, and incident handling — not that you're a full-time DBA writing complex queries.

## ✅ Answer  
Day to day I handle the **operational** side of databases: verifying **backups/snapshots**, **monitoring** performance and storage, managing **access** (IAM, users, secrets), **scaling** (storage autoscaling, read replicas, instance sizing), applying **maintenance/patching**, and responding to **alerts** like high connections or low free storage.

### 📘 Detailed Explanation  

| Area | What I do |
|---|---|
| **Backups & recovery** | Confirm automated snapshots run; test **point-in-time recovery**; copy snapshots cross-region for DR |
| **Monitoring** | CloudWatch dashboards/alarms on CPU, connections, **FreeStorageSpace**, replica lag, read/write latency |
| **Scaling** | Enable **storage autoscaling**, add **read replicas** for read-heavy load, resize instance classes, tune connection pools (**RDS Proxy**) |
| **Access & security** | Least-privilege access, rotate credentials via **Secrets Manager**, restrict via security groups, encryption at rest (KMS) |
| **Maintenance** | Schedule **patching**/maintenance windows, manage **parameter groups**, engine upgrades |
| **Provisioning (IaC)** | Create/modify RDS/DynamoDB via **Terraform**, keep changes reviewed and repeatable |
| **Incident response** | Handle "storage full", "too many connections", slow queries, failover events |

#### 🧩 Typical week
- Check backup/replication health.
- Respond to a **FreeStorageSpace** or **DatabaseConnections** alarm.
- Add a read replica ahead of a launch, or resize an under/over-provisioned instance.
- Rotate a credential and confirm the app picks it up.

---

### 🧠 Real-world Example

> A `DatabaseConnections` alarm fired during peak. The app was opening too many direct connections to RDS. I put **RDS Proxy** in front to pool connections and added a CloudWatch alarm threshold — the connection storms stopped. Routine work like this, plus backup checks and storage autoscaling, is most of my DB day.

> Summary:  
> As a DevOps engineer my DB work is **operational**: backups/PITR, monitoring & alarms, scaling (autoscaling, replicas, RDS Proxy), access/secrets, patching via maintenance windows, IaC provisioning, and incident response — keeping managed databases healthy, secure, and available.
