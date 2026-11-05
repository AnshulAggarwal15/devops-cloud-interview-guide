## Question  
What is the difference between a Resource and a Data Source in Terraform?

### 📝 Short Explanation  
This tests a core concept: a **resource** *creates and manages* infrastructure, while a **data source** only *reads/looks up* existing infrastructure. Mixing them up is a common beginner mistake.

## ✅ Answer  
- A **resource** (`resource` block) **creates, updates, and destroys** infrastructure that Terraform manages.
- A **data source** (`data` block) is **read-only** — it **fetches information** about existing resources (created elsewhere or by another config) so you can reference it, without managing it.

### 📘 Detailed Explanation  

| | Resource | Data Source |
|---|---|---|
| Block | `resource "..." "..." {}` | `data "..." "..." {}` |
| Action | **Creates/manages** infra | **Reads** existing infra |
| Lifecycle | Tracked in state; can be destroyed | Just a lookup; not created/destroyed |
| Use when | You own and manage the resource | You need info about something that already exists |

#### 🧱 Resource — creates something
```hcl
resource "aws_instance" "web" {
  ami           = "ami-0abc123"
  instance_type = "t3.micro"
}
```
Terraform will **create** this instance and manage its lifecycle.

#### 🔍 Data source — reads something
```hcl
# Look up the latest Amazon Linux AMI instead of hardcoding it
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id   # referenced from the data source
  instance_type = "t3.micro"
}
```
Terraform **reads** the AMI ID at plan time; it doesn't create or own the AMI.

#### 🧭 When to use a data source
- Reference an existing **VPC, subnet, AMI, or security group** not managed by this config.
- Pull values shared across teams/stacks (e.g. via `terraform_remote_state`).
- Avoid hardcoding IDs that can change.

---

### 🧠 Real-world Example

> Our networking team owned the VPC. In the app config we used a **data source** (`data.aws_vpc`, `data.aws_subnets`) to **look up** their VPC/subnet IDs, then created our EC2/ALB **resources** inside them. We read what we didn't own, and managed only what we did.

> Summary:  
> **Resource = create & manage** infrastructure (tracked in state, destroyable). **Data source = read-only lookup** of existing infrastructure. Use data sources to reference things you don't own and to avoid hardcoding IDs.
