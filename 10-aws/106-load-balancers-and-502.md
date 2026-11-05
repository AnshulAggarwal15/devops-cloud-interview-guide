<!-- #genai: AWS interview note - ALB/NLB, target groups, and debugging 502/503 for the Docusign R2 cloud round -->

## Question
Compare ALB vs NLB, and walk through how you'd debug users getting 502s behind an ALB.

### Short Explanation
**ALB** is a Layer 7 (HTTP/HTTPS) load balancer with routing rules; **NLB** is Layer 4 (TCP/UDP) built for extreme throughput and static IPs. A **502** from an ALB almost always means the **target (backend) failed the request**, not the ALB itself.

## Answer

| | ALB (Application LB) | NLB (Network LB) |
|---|---|---|
| Layer | 7 (HTTP/HTTPS) | 4 (TCP/UDP/TLS) |
| Routing | Path/host/header rules | By connection, no L7 logic |
| Performance | High | Ultra-high, ultra-low latency |
| IPs | DNS name (dynamic) | Static/Elastic IP support |
| Use | Web apps, microservices, EKS/ECS ingress | High-throughput TCP, low latency, static IP needs |

Traffic path: **Client -> Listener -> Rule -> Target Group -> Target (EC2/pod/IP)**, gated by **health checks**.

### "Where is the load balancer in that path?"
The **Listener, Rules, and Target Groups are components *inside* the load balancer** — they aren't separate hops. The load balancer is the box that *contains* them:

```text
        ┌───────────────── LOAD BALANCER (the ALB) ─────────────────┐
Client →│  Listener (:443)  →  Rule (if /api → TG-api)  →  Target Group │→ Target
        └────────────────────────────────────────────────────────────┘        (EC2 / pod / IP)
                                                                   ▲ health checks decide
                                                                     which targets are eligible
```

- **The load balancer** = the DNS-addressable entry point (what Route 53 points at). It *owns* the listeners.
- **Listener** = "I listen on port 443 (HTTPS)." Terminates TLS. Every LB has one or more.
- **Rule** = *within a listener*, "if host/path matches X, forward to Target Group Y" (ALB only; L7).
- **Target Group** = a named pool of backends (EC2/pod/IP) + the **health check** config for that pool.
- **Target** = the actual backend instance/pod/IP that serves the request.

So there's no missing "load balancer" step — **the Listener/Rule/Target Group *are* the load balancer doing its job**. The one-line version: *"the LB receives on a **listener**, a **rule** picks a **target group**, and it forwards to a **healthy target**."*

## Detailed Explanation

### 502 Bad Gateway vs 503 Service Unavailable
- **502** = ALB reached the target but got an **invalid/failed response** (app crashed, closed the connection, timed out mid-response, bad protocol).
- **503** = **no healthy targets** in the target group to send to.

### Debugging a 502 (say this in order)
1. **Target group health** — are targets healthy? Failing health checks -> being pulled out.
2. **App logs on the target** — is the process crashing / throwing 5xx / OOM?
3. **Timeouts** — app slower than the ALB idle timeout, or keep-alive shorter than ALB's (classic cause).
4. **Security groups** — does the target SG allow the ALB SG on the app port?
5. **Port/protocol mismatch** — target group port vs the port the app actually listens on; HTTP vs HTTPS to target.
6. **ALB access logs / CloudWatch** — `HTTPCode_ELB_5XX` vs `HTTPCode_Target_5XX` tells you ALB-side vs app-side.

### Kubernetes tie-in
Behind EKS, the "target" is your pod/Service. A 502 often maps to a `CrashLoopBackOff`, failing readiness probe, or wrong `targetPort` — same debugging chain, one layer down.

## Summary
- **ALB = L7 (routing), NLB = L4 (throughput/static IP).**
- **502 = target returned a bad/failed response; 503 = no healthy targets.**
- Debug order: target health -> app logs -> timeouts -> SG rules -> port/protocol -> ALB metrics/logs.
