## Question  
Can applications in different subnets of a VPC interact by default? If no, why?

### 📝 Short Explanation  
This checks whether you understand VPC-internal routing and the difference between the **local route** (which enables intra-VPC communication) and the **security controls** (SGs and NACLs) that can still block it.

## ✅ Answer  
**Yes — by default they can.** Every VPC has an implicit **local route** that connects all subnets within the VPC's CIDR, so instances in different subnets can reach each other. Communication only fails if **security groups** or **NACLs** are configured to block it.

### 📘 Detailed Explanation  

#### 🔗 The local route
When you create a VPC, AWS automatically adds a route like:

```
Destination        Target
10.0.0.0/16        local
```

This `local` route covers the **entire VPC CIDR** and cannot be removed. It's what lets a subnet in `10.0.1.0/24` talk to a subnet in `10.0.2.0/24` with no extra configuration.

So **routing is not the blocker** for same-VPC traffic.

#### 🚧 What actually blocks it
If two apps in different subnets can't talk, it's almost always one of:

1. **Security Groups (stateful)** — the target instance's SG doesn't allow inbound from the source SG/IP.
2. **NACLs (stateless)** — a subnet NACL denies the traffic, or allows inbound but forgets the **outbound ephemeral ports** for the return path.
3. Wrong **port / protocol**, or the app isn't listening.

#### 🌍 The exception: different VPCs
Across **separate VPCs**, there is no shared local route — you need **VPC Peering**, **Transit Gateway**, or **PrivateLink** to connect them.

---

### 🧠 Real-world Example

> A web tier in the public subnet couldn't reach the app tier in a private subnet. The route table was fine (local route). The real cause: the app tier's **security group** only allowed port 8080 from the ALB SG, not from the web tier SG. Adding the rule fixed it instantly.

> Summary:  
> **Yes**, subnets in the same VPC talk by default thanks to the **local route**. If they can't, look at **security groups and NACLs**, not routing. Across different VPCs you need peering / TGW / PrivateLink.
