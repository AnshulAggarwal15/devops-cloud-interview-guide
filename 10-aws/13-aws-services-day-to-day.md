## Question  
Which AWS services do you use in your day to day life (as a DevOps/Cloud engineer)?

### 📝 Short Explanation  
This checks that your hands-on experience is **real and broad**. The interviewer wants to hear services grouped by purpose — compute, networking, storage, security, delivery, and observability — with a one-line note on *how* you use each, not just names.

## ✅ Answer  
I group them by what they do: **EC2/EKS/Lambda** for compute, **VPC/ALB/Route 53** for networking, **S3/EBS/EFS/RDS** for storage and data, **IAM/Secrets Manager/KMS** for security, **CloudWatch/CloudTrail** for observability, and **ECR/CodePipeline/CloudFormation** for delivery and IaC.

### 📘 Detailed Explanation  

| Category | Services | How I use them |
|---|---|---|
| **Compute** | EC2, EKS/ECS, Lambda | Run apps, containers, and event-driven jobs |
| **Networking** | VPC, Subnets, ALB/NLB, Route 53, NAT GW | Segment networks, load balance, DNS |
| **Storage & DB** | S3, EBS, EFS, RDS, DynamoDB | Object/block/shared storage, relational & NoSQL |
| **Security & Identity** | IAM, STS, Secrets Manager, KMS | Access control, temp credentials, secrets, encryption |
| **Observability** | CloudWatch, CloudTrail | Metrics, logs, alarms, and API auditing |
| **Delivery & IaC** | ECR, CodePipeline/Jenkins, CloudFormation/Terraform | Image registry, pipelines, infrastructure as code |
| **Cost** | Cost Explorer, Budgets | Track and control spend |

#### 🗣️ How to answer well
- Don't just list — attach a **use case**: "IAM roles for **IRSA** so pods get scoped AWS access," or "**S3 lifecycle** to tier logs to Glacier."
- Mention the ones **central to your role** first (if you run Kubernetes, lead with EKS, ECR, IAM, ALB).

---

### 🧠 Real-world Example

> On a typical day I deploy workloads to **EKS**, pull images from **ECR**, manage access with **IAM roles**, store artifacts and logs in **S3**, watch dashboards and alarms in **CloudWatch**, and provision everything with **Terraform**. Secrets come from **Secrets Manager** via IRSA.

> Summary:  
> Answer by **category with real use cases**: compute (EC2/EKS/Lambda), networking (VPC/ALB/Route 53), storage/data (S3/EBS/EFS/RDS/DynamoDB), security (IAM/STS/Secrets Manager/KMS), observability (CloudWatch/CloudTrail), and delivery/IaC (ECR/pipelines/Terraform).
