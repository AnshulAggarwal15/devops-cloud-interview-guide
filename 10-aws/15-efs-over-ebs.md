## Question  
When will you go for EFS over EBS in real time?

### 📝 Short Explanation  
This tests whether you know the core difference: **EBS is a block volume attached to a single instance**, while **EFS is a shared file system many instances can mount at once**. The choice comes down to **shared access, scale, and I/O pattern**.

## ✅ Answer  
I choose **EFS when multiple instances or pods need to read/write the same files concurrently**, or when I want storage that **auto-scales** and spans AZs. I stick with **EBS when a single instance needs low-latency block storage**, like a database or the OS/root volume.

### 📘 Detailed Explanation  

| | EBS | EFS |
|---|---|---|
| Type | **Block** storage | **File** storage (NFS) |
| Attach | **One instance** (mostly, one AZ) | **Many** instances across **multiple AZs** |
| Scaling | Fixed size you provision | **Elastic**, grows/shrinks automatically |
| Latency | **Lowest** (great for DBs) | Higher than EBS |
| Scope | Single AZ | Regional (multi-AZ) |
| Use case | DB volumes, OS disk, single-node apps | Shared content, multi-node apps, CI, ML data |

#### ✅ Go for EFS when…
- **Many nodes need the same data** (a fleet of web servers sharing uploads, a scaled-out app, shared CI workspace).
- You want **multi-AZ** shared access without managing replication.
- Capacity is unpredictable and you want it to **auto-scale**.

#### ✅ Go for EBS when…
- A **single instance** needs fast, low-latency block storage (databases, boot volume).
- You need consistent high **IOPS** (e.g. `io2`) for a DB.

---

### 🧠 Real-world Example

> A horizontally scaled CMS had 6 web servers that all needed the **same uploaded media**. EBS can't be shared cleanly across instances/AZs, so I used **EFS** mounted on all of them — every server saw the same files instantly. For the CMS's **database**, though, I used a fast **EBS `gp3`/`io2`** volume for low latency.

> Summary:  
> **EFS = shared, multi-AZ, auto-scaling file storage for many clients.** **EBS = single-instance, low-latency block storage** for databases and OS disks. Pick EFS the moment you need concurrent shared access across instances/AZs.
