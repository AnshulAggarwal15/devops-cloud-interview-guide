## Question  
Write Terraform code to create any resource on AWS.

### 📝 Short Explanation  
A hands-on check. The interviewer wants to see that you know the **basic anatomy** of Terraform config: `provider`, `resource`, `variable`, and `output` — and that you can write clean, working HCL from memory.

## ✅ Answer  
I'll show a minimal, complete example that creates an **S3 bucket** (and note an EC2 variant). It has the four building blocks: a **provider**, a **resource**, a **variable** for the name, and an **output** for the result.

### 📘 Detailed Explanation  

#### 🧱 Anatomy of a Terraform config
- **provider** — which cloud/region.
- **resource** — the thing to create.
- **variable** — parameterize inputs.
- **output** — expose useful values.

#### 📄 Example 1: S3 bucket
```hcl
# provider.tf
provider "aws" {
  region = "us-east-1"
}

# variables.tf
variable "bucket_name" {
  description = "Name of the S3 bucket"
  type        = string
}

# main.tf
resource "aws_s3_bucket" "demo" {
  bucket = var.bucket_name

  tags = {
    Environment = "dev"
    ManagedBy   = "terraform"
  }
}

# outputs.tf
output "bucket_arn" {
  value = aws_s3_bucket.demo.arn
}
```

#### 📄 Example 2: EC2 instance
```hcl
resource "aws_instance" "web" {
  ami           = "ami-0abcdef1234567890"
  instance_type = "t3.micro"

  tags = {
    Name = "demo-web"
  }
}
```

#### ▶️ The workflow to apply it
```bash
terraform init      # download providers, set up backend
terraform plan      # preview changes
terraform apply     # create the resource
terraform destroy   # tear it down
```

---

### 🧠 Real-world Example

> In interviews I keep it simple and correct: a `provider`, one `resource`, a `variable` for the name, and an `output`. Then I mention I'd wrap reusable pieces into a **module** and store state in a **remote backend** for real projects — that shows both fundamentals and good practice.

> Summary:  
> A working config needs **provider + resource + variable + output**. Show a clean S3/EC2 example, then run **`init → plan → apply`**. Mention modules and a remote backend to signal production-readiness.
