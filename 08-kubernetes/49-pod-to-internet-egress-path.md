## How a Request Travels From a Pod to the Internet (Pod → google.com on EKS)

### Question  
From a pod in EKS, you hit `google.com`. **How does the request actually travel** from the pod out to the internet and back?

### Short explanation of the question  
This tests whether you understand **Kubernetes networking + AWS networking together**. Two parts: (1) **DNS resolution** (the pod resolves `google.com` via **CoreDNS** → upstream resolver → IP), then (2) the **egress data path** — the **VPC CNI** gives the pod a real VPC IP, the node's **route table** sends `0.0.0.0/0` to a **NAT Gateway** (private subnet) → **Internet Gateway** → internet, and the response returns the same way.

---

### Answer  
1. **Resolve the name (DNS):** the pod asks **CoreDNS** (its `/etc/resolv.conf` nameserver); CoreDNS isn't authoritative for `google.com`, so it **forwards to the upstream resolver** (VPC DNS `169.254.169.253` / configured upstream) → returns Google's IP.
2. **Send the packet (egress path):** the **VPC CNI** gave the pod an IP from the **VPC subnet**, so the packet leaves via the **node's ENI**; the subnet's **route table** decides where `0.0.0.0/0` goes — for a **private subnet** that's a **NAT Gateway** (in a public subnet), which SNATs to a public IP and forwards to the **Internet Gateway** → internet.
3. **Response returns** through IGW → NAT (un-NAT back to the pod IP) → node → pod.

If it fails, check **DNS (CoreDNS/upstream), route table (NAT/IGW), security groups, NACLs, and NetworkPolicies**.

---

### Detailed explanation of the answer for readers’ understanding

### 🧭 The full path (private-subnet EKS, the common case)
```text
Pod
  │ 1. DNS: resolve google.com
  ▼
CoreDNS (kube-system) ── not authoritative ──► forwards to upstream (VPC resolver) ─► returns IP
  │ 2. send HTTPS to that IP
  ▼
VPC CNI  (pod has a real VPC IP) → node ENI
  ▼
Subnet route table   ( 0.0.0.0/0 → nat-xxxx )
  ▼
NAT Gateway  (in a PUBLIC subnet; SNAT: pod IP → NAT's public IP)
  ▼
Internet Gateway (IGW)
  ▼
Internet → Google
        (response returns the same path; NAT translates back to the pod IP)
```

### 🔑 The pieces and their jobs
| Component | Role |
|---|---|
| **CoreDNS** | Resolves the name; **forwards** non-cluster names (`google.com`) to the upstream resolver |
| **VPC CNI** | Assigns the pod a **routable VPC IP** from the subnet (pod traffic is real VPC traffic, via the node ENI) |
| **Route table** | Decides egress: `0.0.0.0/0 → NAT` (private) or `→ IGW` (public) |
| **NAT Gateway** | **Outbound-only** SNAT for private subnets; lets pods reach the internet with no inbound exposure |
| **Internet Gateway** | The VPC's door to the internet |
| **Security Group / NACL** | SG (stateful, on the ENI) + NACL (stateless, subnet) must allow the traffic |
| **NetworkPolicy** | K8s-level egress rules can block pod egress before it ever leaves |

### 🧱 Why NAT (and the private-subnet norm)
- EKS worker nodes/pods usually live in **private subnets** (no public IPs) for security. Outbound internet therefore goes through a **NAT Gateway** in a public subnet → IGW. NAT is **outbound-initiated only**, so nothing on the internet can start a connection to the pod.
- (Public-subnet nodes would route `0.0.0.0/0 → IGW` directly, but that's uncommon for workloads.)

### 🩺 If pod → internet fails, check in order
1. **DNS**: `kubectl exec` → `nslookup google.com` / `cat /etc/resolv.conf`. CoreDNS up? forwarding working? (See [CoreDNS deep dive](./38-coredns-deep-dive-and-troubleshooting.md).)
2. **Route table**: does the pod's subnet have `0.0.0.0/0 → NAT` (and the NAT's subnet `→ IGW`)?
3. **NAT/IGW**: NAT Gateway healthy, in a public subnet, EIP attached?
4. **Security group**: node/pod SG allows outbound 443?
5. **NACL**: subnet NACL allows outbound **and** the ephemeral-port return traffic (stateless!)?
6. **NetworkPolicy**: a default-deny egress policy blocking the pod?

> Classic gotcha: DNS resolves fine but connections hang → usually a **route-table/NAT** or **NACL return-traffic** issue, not DNS.

### ⚠️ Things they probe
- **Two stages: DNS (CoreDNS → upstream) then data path (CNI → ENI → route table → NAT → IGW).**
- **VPC CNI gives pods real VPC IPs** — pod egress is VPC-native, not an overlay hidden behind the node.
- **NAT = outbound-only**, sits in a **public** subnet; private subnets route `0.0.0.0/0 → NAT`.
- **NACLs are stateless** → must allow return traffic on ephemeral ports (a common silent failure).

---

### 🧪 Real-World Example

> "A pod couldn't reach an external API. `nslookup` inside the pod resolved fine, so DNS was OK. The pod's private subnet route table was missing `0.0.0.0/0 → NAT` after a change — so packets had nowhere to go. Adding the NAT route fixed it. I always separate it into **DNS working?** then **egress path (route table/NAT/IGW, SG, NACL)?**"

---

### Key takeaway

> "Two stages: **DNS** — the pod asks **CoreDNS**, which **forwards** `google.com` to the upstream resolver for an IP; then the **data path** — the **VPC CNI** gave the pod a real VPC IP, so the packet exits the **node ENI**, the **route table** sends `0.0.0.0/0` to a **NAT Gateway** (private subnet) → **Internet Gateway** → internet, and returns the same way. Debug order on failure: **DNS → route table/NAT/IGW → SG → NACL (stateless!) → NetworkPolicy**."

### Related
- [CoreDNS deep dive](./38-coredns-deep-dive-and-troubleshooting.md) · [Pod-to-pod / service communication](./04-pod-ip-communication.md) · [Network policies](./12-network-policies.md)
- [VPC, subnets & routing (NAT/IGW)](../10-aws/101-vpc-subnets-routing.md) · [NAT and when it's used](../10-aws/02-aws-nat-and-when-used.md) · [Service communication](../10-aws/110-service-communication.md)
