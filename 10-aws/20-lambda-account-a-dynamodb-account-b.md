## Question  
How does a Lambda function in AWS Account A interact with a DynamoDB table in Account B?

### 📝 Short Explanation  
Another **cross-account** question, but DynamoDB behaves differently from S3: **DynamoDB has no resource-based (table) policy in the classic sense**, so the clean pattern is **STS AssumeRole** — the Lambda assumes a role in Account B and uses temporary credentials.

## ✅ Answer  
The Lambda in Account A **assumes an IAM role in Account B** via **STS**. Account B creates a role with **DynamoDB permissions** and a **trust policy** allowing Account A's Lambda role to assume it. The Lambda calls `sts:AssumeRole`, gets **temporary credentials**, and uses them to call DynamoDB in Account B.

### 📘 Detailed Explanation  

Because DynamoDB access is driven by **IAM (identity) policies**, the assume-role pattern is the standard approach.

#### 🅱️ Account B — create the role
**Permission policy** (what it can do):
```json
{ "Effect": "Allow",
  "Action": ["dynamodb:GetItem","dynamodb:PutItem","dynamodb:Query"],
  "Resource": "arn:aws:dynamodb:us-east-1:B_ACCOUNT_ID:table/Orders" }
```
**Trust policy** (who can assume it) — trust Account A's Lambda role:
```json
{ "Effect": "Allow",
  "Principal": { "AWS": "arn:aws:iam::A_ACCOUNT_ID:role/lambda-exec-role" },
  "Action": "sts:AssumeRole" }
```

#### 🅰️ Account A — let the Lambda assume that role
The Lambda's execution role needs:
```json
{ "Effect": "Allow",
  "Action": "sts:AssumeRole",
  "Resource": "arn:aws:iam::B_ACCOUNT_ID:role/cross-account-dynamo-role" }
```

#### 🔁 In the Lambda code
```python
import boto3
creds = boto3.client("sts").assume_role(
    RoleArn="arn:aws:iam::B_ACCOUNT_ID:role/cross-account-dynamo-role",
    RoleSessionName="lambda-dynamo")["Credentials"]

ddb = boto3.client("dynamodb",
    aws_access_key_id=creds["AccessKeyId"],
    aws_secret_access_key=creds["SecretAccessKey"],
    aws_session_token=creds["SessionToken"])
ddb.get_item(TableName="Orders", Key={"id": {"S": "123"}})
```

---

### 🧠 Real-world Example

> A reporting Lambda in our analytics account had to read an `Orders` table owned by the app account. The app account created a **scoped role** (read-only on that one table) trusting our Lambda role. The Lambda **assumed the role** and queried with **temporary credentials** — no static keys, and the app team kept full control of the permissions.

> Summary:  
> DynamoDB has no bucket-style resource policy, so use **STS AssumeRole**: Account B makes a role with **DynamoDB permissions + a trust policy** for Account A; the Lambda assumes it and uses **temporary credentials**. This keeps access scoped and keyless.
