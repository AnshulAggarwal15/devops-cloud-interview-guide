## CIDR & Subnetting Basics

### Question  
What is **CIDR**? How do you represent a range like a `/16` or `/24` — and a **single IP**? (And how does this apply in AWS?)

### Short explanation of the question  
**CIDR (Classless Inter-Domain Routing)** is just a compact way to write an **IP range**: an address plus a **prefix length** (`/n`) that says how many leading bits are the fixed "network" part. The interviewer usually wants the intuition (bigger `/n` = smaller range), the **`/32` = one IP** trick, and how you use CIDRs for **VPCs, subnets, and allow-lists**.

---

### Answer  
- `A.B.C.D/n` means the **first `n` bits are the network**; the remaining `32 − n` bits vary → that's the range size.
- **Bigger prefix = smaller range.** `/16` = 65,536 addresses, `/24` = 256, **`/32` = exactly one IP**.
- In AWS you use CIDRs to size a **VPC** (`10.0.0.0/16`), carve **subnets** (`10.0.1.0/24`), and write **allow-lists** (a single office IP as `203.0.113.25/32`).

---

### Detailed explanation of the answer for readers’ understanding

### 🔢 How the prefix maps to size
An IPv4 address is **32 bits**. The `/n` fixes the first `n` bits; the rest are host bits → **2^(32−n)** addresses.
| CIDR | Host bits | Total addresses | Typical use |
|---|---|---|---|
| `/16` | 16 | **65,536** | A whole VPC (`10.0.0.0/16`) |
| `/24` | 8 | **256** | A subnet (`10.0.1.0/24`) |
| `/28` | 4 | 16 | Smallest AWS subnet |
| `/30` | 2 | 4 | Point-to-point link |
| `/32` | 0 | **1** | A single host / allow-list entry |

Example: `10.0.0.0/16` covers `10.0.0.0 – 10.0.255.255`. `10.0.1.0/24` covers `10.0.1.0 – 10.0.1.255`.

### 🎯 A single IP = `/32`
To match **exactly one** address, use **`/32`** (0 host bits):
```text
203.0.113.25/32   → only 203.0.113.25
```
That's what you put in a **security group rule** or **bucket policy `aws:SourceIp`** to allow just one office/VPN IP.
```text
/32 → 1 IP     /30 → 4     /28 → 16     /24 → 256     /16 → 65,536
```
(For point-to-point, `/31` = 2 addresses per RFC 3021.)

### ☁️ How AWS uses CIDR
```text
VPC          10.0.0.0/16     # the whole private network
 ├── subnet  10.0.1.0/24     # public-a
 ├── subnet  10.0.2.0/24     # private-a
 └── ...
Allow-list   203.0.113.25/32 # a single external IP in an SG/policy
```
- **Don't overlap** VPC CIDRs you intend to peer or VPN — overlapping ranges break routing.
- **AWS reserves 5 IPs per subnet** (network, VPC router, DNS, future, broadcast), so a `/24` gives **251** usable, not 256.
- **Minimum AWS subnet is `/28`**; VPC CIDR can be `/16`–`/28`. AWS doesn't allow `/31` or `/32` for subnets.
- Leave headroom — with the **EKS VPC CNI**, pods consume **real subnet IPs**, so subnets drain fast.

### ⚠️ Things they probe
- "**Single IP?**" → **`/32`**. Instant answer.
- Bigger `/n` = **fewer** addresses (people invert this).
- AWS subnet quirks: **5 reserved IPs**, **min `/28`**, no overlap for peering.

---

### Key takeaway

> "**CIDR = IP + prefix length**; the prefix says how many bits are the network, so **bigger `/n` = smaller range** (`/16` = 65,536, `/24` = 256, **`/32` = one IP**). In AWS: `/16` for a VPC, `/24` for subnets, **`/32` for a single-IP allow-list** — mind the **5 reserved IPs per subnet** and avoid overlapping CIDRs."

### Related
- [VPC & subnet design](../10-aws/116-vpc-and-subnet-design.md) · [VPC, subnets & routing](../10-aws/101-vpc-subnets-routing.md)
- [IP whitelisting](../10-aws/117-ip-whitelisting.md)
