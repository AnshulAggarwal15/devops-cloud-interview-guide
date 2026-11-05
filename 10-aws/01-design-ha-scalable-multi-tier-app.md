## Question  
Explain how will you design a highly available and scalable multi-tier application on AWS.

### 📝 Short Explanation  
This question checks whether you can translate the classic 3-tier application (web → app → database) into AWS building blocks while keeping it **highly available** (survives an AZ failure) and **scalable** (grows and shrinks with load). The interviewer wants to hear about Multi-AZ, load balancing, auto scaling, and a managed database.

## ✅ Answer  
I spread every tier across **at least two Availability Zones**, put the app behind an **Application Load Balancer**, run the compute in an **Auto Scaling Group**, and use a **Multi-AZ managed database (RDS)**. Static content goes to **S3 + CloudFront**, and I keep the tiers isolated using public and private subnets.

### 📘 Detailed Explanation  

Think of the request flowing top to bottom, and make each layer redundant.

#### 1. 🌐 Edge and DNS
- **Route 53** for DNS, with health checks so traffic only goes to healthy endpoints.
- **CloudFront** in front of static assets (and optionally the app) to cache content close to users.

#### 2. ⚖️ Web / presentation tier
- An **Application Load Balancer (ALB)** in **public subnets across 2+ AZs**.
- The ALB spreads traffic across the app instances and removes unhealthy ones automatically.

#### 3. 🖥️ Application tier
- EC2 instances (or containers on **ECS/EKS**) in **private subnets across 2+ AZs**.
- Wrapped in an **Auto Scaling Group** with scaling policies (e.g. target 60% CPU) so it adds instances under load and removes them when idle.
- No public IPs — reachable only through the ALB.

#### 4. 🗄️ Database tier
- **Amazon RDS with Multi-AZ** for automatic failover to a standby in another AZ.
- **Read replicas** to scale read-heavy traffic.
- Lives in private subnets; only the app tier's security group can reach it.

#### 5. 📦 Storage and state
- Keep the app **stateless** — store sessions in **ElastiCache/DynamoDB** and files in **S3**, so any instance can serve any request.

---

### 🧠 Design in one picture

```
Route 53 → CloudFront → ALB (public subnets, 2 AZs)
                             │
                    App ASG (private subnets, 2 AZs)
                             │
                    RDS Multi-AZ (private subnets, 2 AZs)
   Static assets → S3 + CloudFront   Sessions → ElastiCache
```

---

### 🔒 Don't forget
- **Security groups** per tier (ALB → app → db), least privilege.
- **NAT Gateway** (one per AZ) so private instances can reach the internet for updates.
- **CloudWatch** alarms driving the scaling policies.

> Summary:  
> HA comes from **spreading every tier across multiple AZs** (ALB, ASG, RDS Multi-AZ); scalability comes from the **Auto Scaling Group + stateless app + managed data services**. Public subnets for the ALB, private subnets for app and DB.
