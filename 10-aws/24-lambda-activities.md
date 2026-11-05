## Question  
Have you worked with Lambda in AWS? Explain your activities with Lambda.

### 📝 Short Explanation  
An experience question about **serverless**. The interviewer wants concrete use cases where you used Lambda for **automation and event-driven** work — not a definition of what Lambda is.

## ✅ Answer  
Yes. I use Lambda mainly for **event-driven automation and glue tasks**: reacting to **S3/SNS/SQS/EventBridge** events, running **scheduled jobs** (cron-style cleanup, snapshots), sending **notifications/alerts**, and small integrations between services — all without managing servers.

### 📘 Detailed Explanation  

#### 🔧 Common things I've built with Lambda

| Use case | Trigger | What it does |
|---|---|---|
| **Scheduled cleanup** | EventBridge (cron) | Delete old EBS snapshots, expire temp S3 objects, stop idle non-prod EC2 |
| **S3 processing** | S3 event | Validate/transform uploaded files, kick off a workflow |
| **Notifications** | SNS / CloudWatch alarm | Send Slack/email on alarms or pipeline events |
| **Automation glue** | EventBridge rules | Auto-tag new resources, react to CloudTrail events |
| **Cross-account access** | Invoked by app | Assume a role via STS and act in another account |

#### ⚙️ Operational things I care about
- **IAM execution role** with least privilege.
- **Timeout & memory** tuned to the workload (memory also scales CPU).
- **Concurrency limits** and **DLQ**/destinations so failures aren't lost.
- **CloudWatch Logs/metrics** for observability, and cold-start awareness for latency-sensitive functions (**provisioned concurrency** when needed).
- Deployed via **IaC** (Terraform/SAM) and CI/CD, not the console.

---

### 🧠 Real-world Example

> We had EBS snapshot sprawl inflating the bill. I wrote a **Lambda on an EventBridge cron** that deletes snapshots older than the retention policy and Slack-notifies a summary. Its execution role was scoped to just the snapshot APIs, and it logged to CloudWatch. Fully serverless, ran nightly, and cut snapshot cost noticeably.

> Summary:  
> I use Lambda for **event-driven automation**: S3/SNS/SQS/EventBridge triggers, **scheduled cleanup jobs**, notifications, and cross-account glue — with a least-privilege role, tuned timeout/memory, a DLQ, CloudWatch observability, and IaC-based deployment.
