## Question  
How can an AWS Lambda in one AWS account connect to an S3 bucket in another account?

### 📝 Short Explanation  
This is a classic **cross-account access** question. The interviewer wants to see that you know access needs **both sides to agree**: the Lambda's **IAM role** (identity policy) *and* the bucket's **resource policy** (bucket policy). Two approaches exist — bucket policy or role assumption.

## ✅ Answer  
There are two ways:
1. **Bucket policy (resource-based):** the S3 bucket in Account B grants access to the Lambda's **execution role** in Account A, and the role is allowed the S3 actions. No role switching needed.
2. **Assume role (cross-account role):** Account B creates a role that trusts Account A; the Lambda calls **STS AssumeRole** to get temporary credentials, then accesses the bucket.

### 📘 Detailed Explanation  

#### 🅰️ Approach 1 — Bucket policy (simplest for S3)

**Account A (Lambda side):** the execution role allows S3 actions.
```json
{ "Effect": "Allow",
  "Action": ["s3:GetObject","s3:PutObject"],
  "Resource": "arn:aws:s3:::account-b-bucket/*" }
```

**Account B (bucket side):** the bucket policy trusts Account A's role.
```json
{ "Effect": "Allow",
  "Principal": { "AWS": "arn:aws:iam::ACCOUNT_A_ID:role/lambda-exec-role" },
  "Action": ["s3:GetObject","s3:PutObject"],
  "Resource": "arn:aws:s3:::account-b-bucket/*" }
```
Both must allow — S3 evaluates identity **and** resource policy.

> ⚠️ Watch **object ownership/ACLs**: if Account A writes objects, enable **Bucket owner enforced / bucket-owner-full-control** so Account B still owns/reads them.

#### 🅱️ Approach 2 — Assume a role in Account B
- Account B creates a role with an S3 policy and a **trust policy** allowing Account A to assume it.
- The Lambda calls `sts:AssumeRole` → gets temporary creds → uses them for S3.
- Preferred when Account B wants to keep control and hand out a **scoped role** rather than editing the bucket policy.

---

### 🧠 Real-world Example

> A data-processing Lambda in our app account needed to read a partner's S3 bucket in another account. We used a **bucket policy** granting our **Lambda execution role** `s3:GetObject`, and set **bucket-owner-enforced** so ownership stayed clean. No STS hop needed for a simple read.

> Summary:  
> Cross-account S3 needs **both** sides to agree. Easiest: **bucket policy** grants the Lambda's **execution role** (plus the role's own S3 permission). Alternative: **STS AssumeRole** into a role in Account B. Mind **object ownership** when writing across accounts.
