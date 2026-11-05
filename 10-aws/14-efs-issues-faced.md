## Question  
Have you used AWS EFS? If yes, what issues did you run into?

### 📝 Short Explanation  
**EFS (Elastic File System)** is a managed, **shared NFS** file system that many instances/pods can mount at once. The interviewer wants to confirm real usage by hearing the **practical pain points** — mount/security-group issues, latency, throughput modes, and cost.

## ✅ Answer  
Yes. EFS is great when **multiple instances/pods need the same files** concurrently. The issues I've hit are: **mount failures** due to security groups/DNS, **higher latency** than EBS for small random I/O, **throughput throttling** in Bursting mode, and **cost** growing quietly for infrequently accessed data.

### 📘 Detailed Explanation  

#### ✅ Where EFS fits
- Shared storage across **many** instances/pods (EBS attaches to one AZ/one instance at a time).
- Content management, shared uploads, CI workspaces, ML data shared by workers.
- Auto-scales capacity; pay for what you use.

#### ⚠️ Real issues I ran into

| Issue | Cause | Fix |
|---|---|---|
| **Mount hangs / fails** | Port **2049 (NFS)** blocked by SG, or mount target missing in the AZ | Allow 2049 between instance SG and mount-target SG; create a mount target **per AZ** |
| **Slow small-file I/O** | EFS latency > EBS for random small reads | Use EBS for latency-sensitive DBs; EFS for shared/sequential |
| **Throughput throttling** | **Bursting** mode ran out of burst credits | Switch to **Elastic/Provisioned throughput** |
| **Cost creep** | Cold data staying in Standard | Enable **EFS Lifecycle** → move to **IA** tier |
| **Permissions** | UID/GID mismatch across clients | Standardize POSIX perms / use **access points** |

#### 🔌 Mounting (typical)
```bash
# NFS client mount
sudo mount -t nfs4 -o nfsvers=4.1 \
  fs-xxxx.efs.<region>.amazonaws.com:/ /mnt/efs
```
In Kubernetes I used the **EFS CSI driver** with a `PersistentVolume` backed by the file system.

---

### 🧠 Real-world Example

> Pods couldn't mount EFS and the mount just hung. Root cause: the EFS **mount target's security group** didn't allow **NFS port 2049** from the node security group. Opening 2049 fixed it. Later we hit **burst throughput limits** during batch jobs and switched to **Elastic throughput**.

> Summary:  
> EFS = **shared NFS for many clients**. Watch for **SG/port 2049 mount issues**, **higher latency than EBS** for small random I/O, **throughput mode throttling**, and **cost** — solved with lifecycle IA tiering and the right throughput mode.
