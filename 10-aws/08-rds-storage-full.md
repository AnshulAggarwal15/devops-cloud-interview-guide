## AWS RDS Storage is Full — Troubleshooting

### Question  
What will you do when AWS RDS storage is full?

### Short explanation of the question  
A full RDS volume can make the database **read-only or unavailable**, so the interviewer wants both an **immediate mitigation** and a **permanent prevention** (autoscaling storage + monitoring), plus knowledge of what commonly eats the space.

### Answer  
Short term, I **scale up the allocated storage** (RDS allows storage increase without downtime) and free up space by clearing logs/old data. Long term, I enable **Storage Autoscaling**, set **CloudWatch alarms on FreeStorageSpace**, and fix whatever is consuming disk (logs, binlogs, temp files, unbounded tables).

### Detailed explanation of the answer

#### 1. 🚑 Immediate mitigation — increase storage
- Modify the instance and **increase Allocated Storage**. RDS applies this **online** (no downtime) for most engines.
```bash
aws rds modify-db-instance \
  --db-instance-identifier mydb \
  --allocated-storage 200 \
  --apply-immediately
```

#### 2. 🧹 Reclaim space
- Purge or archive **old logs** and **binary logs** (reduce the binlog/backup retention if safe).
- Drop unused indexes, clean up large temp tables.
- For MySQL, watch out for a huge `ibdata`/binlog; for Postgres, watch **WAL** and dead tuples (run `VACUUM`).

#### 3. 🔍 Find what filled it
- Check **CloudWatch → FreeStorageSpace** to see how fast it dropped.
- Query the DB for the largest tables/indexes.
- Common culprits: verbose logging, retained binlogs, an unbounded audit/events table, or a runaway job.

#### 4. 🛡️ Prevent recurrence
- Enable **RDS Storage Autoscaling** (set a max limit) so it grows automatically.
- Add a **CloudWatch alarm** on `FreeStorageSpace` (e.g. alert < 10%).
- Set sensible **log/backup retention** and add data lifecycle/archival.

---

### 🧠 Real-world Example

> A production MySQL RDS hit 100% and went read-only during a release. We immediately **bumped allocated storage** (online), then found **binlogs retained for 30 days** were the cause. We cut retention to 3 days and turned on **storage autoscaling** with an alarm — it never happened again.

### Key takeaway

> "Fix it in two moves: **increase storage now** (RDS does it without downtime) and **enable storage autoscaling + FreeStorageSpace alarms** so it never surprises you again. Then hunt down the log/binlog/table that ate the disk."
