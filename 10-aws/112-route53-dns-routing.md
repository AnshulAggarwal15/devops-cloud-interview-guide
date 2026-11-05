<!-- #genai: AWS interview note - Route 53 record types, routing policies, health checks, alias vs CNAME for the cloud round -->

## Question
Explain Amazon Route 53 — record types, routing policies, health checks, and alias vs CNAME.

### Short Explanation
Route 53 is AWS's **managed DNS** and **domain registrar**. The two things interviewers dig into are the **routing policies** (how it decides which answer to return) and **alias vs CNAME** (the AWS-specific record that CNAME can't do).

## Answer
- Route 53 resolves your domain to endpoints and can make **traffic decisions** using **routing policies**.
- **Alias records** are AWS-only records that point a name (including the **zone apex**, e.g. `example.com`) directly at AWS resources (ALB, CloudFront, S3, another Route 53 record) — free of extra lookups, unlike a **CNAME** which can't sit at the apex.
- **Health checks** let Route 53 route away from unhealthy endpoints (failover/latency/weighted).

## Detailed Explanation

### Common record types
| Record | Points to |
|---|---|
| **A / AAAA** | IPv4 / IPv6 address |
| **CNAME** | Another domain name (**not** allowed at the zone apex) |
| **Alias** | AWS resource or another R53 record — **works at the apex**, no charge for the lookup |
| **MX / TXT / NS / SOA** | Mail, verification/SPF/DKIM, delegation, zone metadata |

### Routing policies (the main topic)
| Policy | Use it for |
|---|---|
| **Simple** | One record, one answer |
| **Weighted** | Split traffic by % (canary, blue/green, A/B) |
| **Latency** | Send users to the **lowest-latency** region |
| **Failover** | Primary → **secondary** on health-check failure (active/passive DR) |
| **Geolocation** | Route by **user location** (compliance, localized content) |
| **Geoproximity** | Route by geographic distance with a **bias** to shift traffic |
| **Multivalue answer** | Return several healthy IPs (simple client-side spread + health checks) |

### Alias vs CNAME (very common question)
- **CNAME**: maps a name to another name, but **cannot** be used at the **zone apex** (`example.com`), only subdomains (`www.example.com`).
- **Alias**: an AWS extension that **can** live at the apex, resolves to the resource's IPs automatically, and incurs **no extra query charge**. Use Alias for ALB, CloudFront, S3 website, API Gateway, etc.

### Health checks
- Monitor an endpoint (HTTP/HTTPS/TCP) or a **CloudWatch alarm**.
- Combine with **failover** or **weighted/latency** policies so Route 53 stops returning dead endpoints.
- Can be **calculated** (parent check of multiple child checks) for “is the whole stack healthy?”

### Things they probe
- Route 53 is a **global** service (no region for the hosted zone).
- **TTL** controls client caching — lower TTL = faster failover but more queries; Alias to AWS resources tracks changes automatically.
- Private **hosted zones** resolve names **inside a VPC** only.

## Summary
- Route 53 = managed DNS + registrar + **traffic routing**.
- Policies: **simple, weighted, latency, failover, geolocation, geoproximity, multivalue** — know when to use each.
- **Alias** works at the **zone apex** and points at AWS resources for free; **CNAME** cannot sit at the apex.
- **Health checks** drive failover and remove unhealthy endpoints.

## Related
- [DNS fundamentals](../04-networking/01-explain-dns.md)
- [Load balancers & 502s](./106-load-balancers-and-502.md)
