<!-- #genai: AWS interview note - high availability, multi-AZ, auto scaling, RDS for the Docusign R2 cloud round -->

## Question
How do you design for high availability on AWS, and what changes when traffic suddenly spikes 10x?

### Short Explanation
High availability = **no single point of failure**: spread stateless compute across **multiple Availability Zones** behind a load balancer, scale it with an **Auto Scaling Group**, and use **Multi-AZ** managed data stores.

## Answer
- **Multi-AZ** everything critical: subnets, ASG instances, RDS, NAT.
- **Stateless app tier** behind an **ALB** so any instance handles any request.
- **Auto Scaling Group** with health checks to replace failed instances and scale on demand.
- **RDS Multi-AZ** (standby failover) + **read replicas** for read scaling.

## Detailed Explanation

### The HA building blocks
- **AZs** = physically separate data centers in a region. Run in **>= 2 AZs**; if one fails, others serve.
- **ALB** spans AZs and only routes to **healthy** targets.
- **ASG** keeps desired capacity, replaces unhealthy nodes, and scales via policies (target tracking on CPU/RPS, or scheduled).
- **RDS Multi-AZ** = synchronous standby in another AZ, automatic failover (HA, *not* read scaling). **Read replicas** = async copies for read-heavy load (scaling, not failover).

### Traffic spikes 10x — what changes (say this progression)
1. **Auto scaling** adds instances/pods (ASG target tracking, or HPA/Cluster Autoscaler on EKS).
2. **Cache** hot reads (ElastiCache/Redis, CloudFront) to shield the DB — the usual bottleneck.
3. **DB read replicas** for read-heavy load; **queue (SQS)** to absorb write bursts and smooth spikes.
4. **CDN** offloads static content at the edge.
5. Confirm **limits/quotas** and connection pools aren't the ceiling.

### Instinct they want
"Scale the **stateless** tier horizontally first; protect the **stateful** DB with caching, replicas, and async writes. The database is almost always the bottleneck."

## Summary
- HA = multi-AZ + stateless tier behind ALB + ASG health/replace + Multi-AZ RDS.
- **RDS Multi-AZ = failover/HA; read replicas = read scaling** (don't confuse them).
- 10x spike: autoscale compute -> cache -> read replicas/queue -> CDN; the DB is the bottleneck.
