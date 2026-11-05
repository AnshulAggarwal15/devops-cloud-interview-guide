<!-- #genai: AWS interview note - core services cheat sheet, Secrets Manager vs Parameter Store, cost optimization for the Docusign R2 cloud round -->

## Question
Name the core AWS services you'd use on a platform, the difference between Secrets Manager and Parameter Store, and how you'd optimize cost.

### Short Explanation
Know the workhorse services by category, when to use **Secrets Manager** vs **SSM Parameter Store**, and the standard levers for reducing spend.

## Answer

### Core services by category
| Category | Service |
|---|---|
| Compute | EC2, ECS, EKS, Lambda, Fargate |
| Networking | VPC, Route 53 (DNS), ALB/NLB, CloudFront (CDN) |
| Storage | S3 (object), EBS (block), EFS (shared file) |
| Database | RDS/Aurora (SQL), DynamoDB (NoSQL), ElastiCache (Redis) |
| Registry | ECR (containers) |
| Observability | CloudWatch (metrics/logs/alarms), X-Ray (traces) |
| Security/Config | IAM, KMS, Secrets Manager, SSM Parameter Store |
| Messaging | SQS (queue), SNS (pub/sub), EventBridge |

## Detailed Explanation

### Secrets Manager vs SSM Parameter Store
| | Secrets Manager | Parameter Store (SSM) |
|---|---|---|
| Purpose | Secrets (DB creds, API keys) | Config + secrets (`SecureString`) |
| Rotation | **Built-in automatic rotation** | No native rotation |
| Cost | Per secret + per API call | Standard tier **free** |
| When | Rotating credentials, cross-account secret sharing | Plain config, cheap/simple secrets |
- Rule of thumb: **rotation or managed DB creds -> Secrets Manager**; **cheap config/simple secrets -> Parameter Store**. Both encrypt with **KMS**.

### Cost optimization levers
- **Right-size** instances; use **Auto Scaling** so you don't over-provision.
- **Savings Plans / Reserved Instances** for steady baseline; **Spot** for fault-tolerant/batch.
- **S3 lifecycle** policies (move to IA/Glacier, expire old data); tier logs.
- Delete **unattached EBS volumes / idle Elastic IPs / old snapshots**.
- **NAT Gateway** is a sneaky cost (per-hour + per-GB) — use VPC endpoints for S3/DynamoDB to avoid NAT for that traffic.
- **Tagging** + Cost Explorer / budgets for visibility and accountability.

## Summary
- Know services by category (compute/network/storage/db/observability/security/messaging).
- **Secrets Manager = rotation + managed secrets; Parameter Store = cheap config/secrets.**
- Cost: right-size + autoscale, Savings Plans/Spot, S3 lifecycle, kill idle resources, watch NAT, tag everything.
