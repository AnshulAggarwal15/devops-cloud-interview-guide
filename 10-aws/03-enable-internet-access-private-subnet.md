## Question  
How will you enable internet access to an application deployed in a private subnet?

### 📝 Short Explanation  
This is a very common scenario question. The trick is to separate the **two directions** of "internet access": letting the app **reach out** (outbound) vs letting users **reach the app** (inbound). They need different solutions.

## ✅ Answer  
It depends on the direction:
- **Outbound** (app needs to call the internet): route the private subnet through a **NAT Gateway** in a public subnet.
- **Inbound** (users need to reach the app): put a **load balancer (ALB/NLB) in a public subnet** in front of the app, or use CloudFront — never give the private instance a public IP.

### 📘 Detailed Explanation  

#### 🔷 Case 1: The app must make outbound calls
Example: pull packages, hit an external API.
- Create a **NAT Gateway** in a **public subnet** (which routes to an Internet Gateway).
- Edit the **private subnet's route table**: `0.0.0.0/0 → nat-gateway-id`.
- Now the app can initiate outbound traffic; the internet still cannot start a connection to it.

#### 🔶 Case 2: Users must reach the app
Example: a web app that should be publicly accessible but must stay in a private subnet for security.
- Deploy an **Application Load Balancer in public subnets** across 2+ AZs.
- The ALB's **target group** points to the private instances.
- Users hit the ALB's public DNS; the ALB forwards to the private app over the internal network.
- The instance keeps **no public IP** and only accepts traffic from the **ALB's security group**.

---

### 🧭 The full picture

```
Inbound:  User → Internet Gateway → ALB (public subnet) → App (private subnet)
Outbound: App (private subnet) → NAT Gateway (public subnet) → Internet Gateway → Internet
```

---

### 🧠 Real-world Example

> A payments service must not be directly exposed but has to serve HTTPS and also call a KYC API. I fronted it with an **ALB in public subnets** for inbound, and added a **NAT Gateway** for its outbound KYC calls. The instances themselves stayed fully private.

> Summary:  
> Outbound → **NAT Gateway** + private route table. Inbound → **public load balancer** in front of the private app. Keep the instance itself private in both cases.
