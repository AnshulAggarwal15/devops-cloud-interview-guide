<!-- #genai: AWS interview note - RDS Multi-AZ vs read replicas, backups, scaling, failover for the cloud round -->

## Question
Explain Amazon RDS — Multi-AZ vs read replicas, how backups work, and how you scale it.

### Short Explanation
RDS is a **managed relational database** (MySQL, Postgres, MariaDB, Oracle, SQL Server, plus Aurora). Interviewers mainly want the one thing people confuse: **Multi-AZ = high availability/failover**, **read replicas = read scaling**. They are not the same feature.

## Answer
- **Multi-AZ** keeps a **synchronous standby** in another AZ and **fails over automatically** — it is for **availability**, not performance.
- **Read replicas** are **asynchronous** copies you send **read traffic** to — they are for **scaling reads**, not failover.
- **Backups**: automated daily snapshots + transaction logs give **point-in-time recovery (PITR)**; manual snapshots live until you delete them.
- **Scaling**: vertical (bigger instance class), storage autoscaling, and horizontal reads via replicas. Writes still go to a single primary (unless you use Aurora/sharding).

## Detailed Explanation

### Multi-AZ vs read replicas (the classic trap)

| | Multi-AZ | Read replica |
|---|---|---|
| Purpose | **HA / failover** | **Read scaling** |
| Replication | **Synchronous** | **Asynchronous** |
| Can you read from it? | ❌ No (standby is passive) | ✅ Yes |
| Failover | **Automatic** (DNS flips to standby) | Manual **promote** to standalone primary |
| Cross-region? | Same region (AZs) | ✅ Can be cross-region |
| Endpoint | Same DB endpoint | Separate reader endpoint |

> Aurora blends both: one writer + up to 15 readers sharing storage, with reader endpoints and fast failover.

### Backups & recovery
- **Automated backups**: daily snapshot + 5-minute transaction logs → **PITR** within the retention window (1–35 days).
- **Manual snapshots**: retained until explicitly deleted; survive instance deletion.
- Snapshots can be **copied cross-region** and **shared** across accounts for DR.
- Restoring a snapshot always creates a **new instance** (new endpoint) — plan the cutover.

### Scaling levers
- **Vertical**: change instance class (e.g. `db.t3.large` → `db.r6g.xlarge`) — brief failover/restart.
- **Storage autoscaling**: grows allocated storage automatically (set a max) — online, no downtime.
- **Read scaling**: add **read replicas** and point read-heavy traffic at them; cache hot reads in **ElastiCache** first.
- **Write scaling**: single primary is the ceiling → use **Aurora**, larger instances, or app-level sharding.

### Things they probe
- Multi-AZ failover swaps the **DNS endpoint** to the standby — keep app **connection retries/short DNS TTL** so it reconnects fast.
- Read replicas have **replication lag** → don't read-your-own-write from a replica.
- Encryption at rest (KMS) must be set at **creation**; enabling later means snapshot → restore encrypted.

## Summary
- **Multi-AZ = automatic failover / HA (synchronous standby, not readable).**
- **Read replicas = read scaling (asynchronous, readable, can be cross-region).**
- Backups: automated → **PITR**; manual snapshots persist; copy cross-region for DR.
- Scale up (instance/storage) + scale reads (replicas + cache); writes are bound to one primary unless Aurora/sharding.

## Related
- [High availability & autoscaling](./107-high-availability-and-autoscaling.md)
- [RDS storage full — troubleshooting](./08-rds-storage-full.md)
- [Day-to-day database activities](./23-day-to-day-database-activities.md)
