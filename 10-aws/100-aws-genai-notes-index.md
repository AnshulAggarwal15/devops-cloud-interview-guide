<!-- #genai: index for the 10-aws interview notes (Docusign R2 cloud round focus) -->

# 10 - AWS

Interview notes focused on the topics that come up for a Cloud/Platform Engineer round: networking, IAM, containers, load balancing, HA, and cost. Pairs with `docusign-cloud-devops-1hr-cram.md` Section 1 and the `04-networking` fundamentals.

## Notes
1. [VPC, subnets & routing](./101-vpc-subnets-routing.md) — VPC, public vs private subnets, route tables, IGW, NAT
2. [Security Groups vs NACLs](./102-security-groups-vs-nacls.md) — stateful vs stateless, allow/deny, evaluation order
3. [IAM users, roles & policies](./103-iam-users-roles-policies.md) — trust vs permission policy, policy evaluation, assume-role
4. [IRSA](./104-irsa.md) — how an EKS pod gets AWS credentials securely
5. [ECS vs EKS](./105-ecs-vs-eks.md) — differences and when to choose which
6. [Load balancers & 502s](./106-load-balancers-and-502.md) — ALB vs NLB, target groups, debugging 502/503
7. [High availability & autoscaling](./107-high-availability-and-autoscaling.md) — multi-AZ, ASG, RDS, 10x traffic
8. [Core services, secrets & cost](./108-core-services-secrets-and-cost.md) — service cheat sheet, Secrets Manager vs Parameter Store, cost levers
9. [IAM AccessDenied troubleshooting](./109-iam-access-denied-troubleshooting.md) — scenario walkthrough
10. [Service communication](./110-service-communication.md) — communication patterns and network path between AWS workloads
11. [RDS — Multi-AZ, replicas & scaling](./111-rds-multiaz-replicas-scaling.md) — Multi-AZ (failover) vs read replicas (scaling), backups/PITR, scaling levers
12. [Route 53 — DNS & routing](./112-route53-dns-routing.md) — record types, routing policies, health checks, alias vs CNAME
13. [S3 buckets & security](./113-s3-buckets-security.md) — storage classes, lifecycle, encryption, bucket policy vs IAM, Block Public Access
14. [Amazon MSK (managed Kafka)](./114-msk-managed-kafka.md) — topics/partitions/consumer groups, replication, provisioned vs serverless, vs SQS/SNS/Kinesis
15. [ASG & scaling](./115-asg-and-scaling.md) — Auto Scaling Groups, scaling policies, horizontal vs vertical
16. [VPC & subnet design](./116-vpc-and-subnet-design.md) — CIDR/subnet tiers, peering vs Transit Gateway, VPC endpoints/PrivateLink
17. [IP whitelisting](./117-ip-whitelisting.md) — SG vs NACL vs WAF vs policy conditions, prefix lists
18. [Cloud security fundamentals](./118-cloud-security-fundamentals.md) — shared responsibility, defense in depth, least privilege, encryption, detection
19. [Vault — secrets management](./119-vault-secrets-management.md) — auth methods, secret engines, dynamic secrets, vs Secrets Manager
20. [EKS migration](./120-eks-migration.md) — blue/green cluster cutover, stateless-first, data migration, version upgrades
21. [GitHub Actions OIDC → AWS federation](./26-github-actions-oidc-federation.md) — external systems assuming an IAM role via OIDC, trust policy, `AssumeRoleWithWebIdentity`, JWKS validation
22. [Pod node-role / IMDS troubleshooting](./27-pod-node-role-imds-troubleshooting.md) — pod without IRSA falling back to the node role; IMDSv2 hop-limit/token debugging
23. [Cloud security scenarios](./28-cloud-security-scenarios.md) — secrets-in-image, public-vs-IP buckets, egress exfiltration (preventive + detective), CDN origin bypass, IaC scanning gates
24. [Authentication vs Authorization framework](./29-authentication-vs-authorization.md) — the "who are you? / what can you do?" lens for any "how is it secure?" question; 401 vs 403
25. [EKS from scratch (Terraform)](./30-eks-from-scratch-terraform.md) — layered secure build order, private API endpoint, node groups, IAM/IRSA, add-ons, "increase the size of what?"

## Related
- Kubernetes RBAC: [../08-kubernetes/36-rbac.md](../08-kubernetes/36-rbac.md)
- CI/CD from scratch + GitOps deep dive: [../05-cicd/17-cicd-from-scratch-gitops-deep-dive.md](../05-cicd/17-cicd-from-scratch-gitops-deep-dive.md)
- Networking fundamentals: [../04-networking](../04-networking) (DNS, OSI, proxies, 502, subnets)
- Kubernetes: [../08-kubernetes](../08-kubernetes)
- Cram: [../docusign-cloud-devops-1hr-cram.md](../docusign-cloud-devops-1hr-cram.md) Section 1
