# 08 — AWS Cloud (Advanced)

## 1. AWS հիմնական ծառայություններ (recap)

System Engineer-ի համար ամենակարևորները՝

| Ծառայություն | Նշանակություն |
|--------------|----------------|
| **EC2** | Virtual servers |
| **VPC** | Isolated network (subnet, route table, IGW, NAT) |
| **S3** | Object storage |
| **IAM** | Users, roles, policies |
| **RDS** | Managed relational DB |
| **ELB/ALB** | Load balancing |
| **Route 53** | DNS |
| **CloudWatch** | Metrics, logs, alarms |
| **ECS/EKS** | Containers |
| **Lambda** | Serverless functions |

---

## 2. VPC — advanced

```text
VPC 10.0.0.0/16
├── Public subnet  10.0.1.0/24  → IGW → internet
├── Public subnet  10.0.2.0/24
├── Private subnet 10.0.10.0/24 → NAT Gateway → outbound only
└── Private subnet 10.0.11.0/24 → RDS, internal apps
```

**Կարևոր հասկացություններ՝**

- **Security Group** — stateful firewall (instance level)
- **NACL** — stateless subnet level
- **Route table** — որտեղ է traffic-ը գնում
- **NAT Gateway** — private subnet-ից internet (update, API calls), inbound direct չի
- **VPC Peering / Transit Gateway** — մի VPC-ից մյուսը

Interview-ում հ часто հարցնում են՝ «private EC2-ից ինչպես update անել» → NAT կամ VPC endpoints։

---

## 3. IAM — roles vs users vs policies

```text
User (long-term keys)     → մարդ / CI user (պակաս նախընտրելի production-ում)
Role (temporary creds)    → EC2, Lambda, EKS pod — assume role
Policy (JSON)             → allow/deny actions on resources
```

**Best practices՝**

- EC2/Lambda/EKS — **instance/task role**, ոչ static access keys
- Least privilege
- MFA root account-ի համար
- `sts:AssumeRole` cross-account

Օրինակ policy fragment՝

```json
{
  "Effect": "Allow",
  "Action": ["s3:GetObject"],
  "Resource": "arn:aws:s3:::my-bucket/*"
}
```

---

## 4. High availability և scaling

- **Multi-AZ** — RDS, NAT (optional), ALB across AZs
- **Auto Scaling Group (ASG)** — min/desired/max, health checks, launch template
- **ALB** — target groups, health check path, sticky sessions
- **Route 53** — weighted, failover, latency routing

```text
Route 53 → ALB → ASG (EC2 in 2+ AZ) → RDS Multi-AZ
```

---

## 5. S3 advanced

- **Versioning** — accidental delete protection
- **Lifecycle** — transition to Glacier, expire old objects
- **Encryption** — SSE-S3, SSE-KMS, bucket policy
- **Block Public Access** — default ON
- **Presigned URL** — temporary access without public bucket

Static site + CloudFront CDN — interview scenario։

---

## 6. RDS և backups

- **Multi-AZ** — synchronous standby (failover)
- **Read replica** — read scaling, async
- **Snapshots** — manual + automated backup window
- **Parameter group** — engine tuning
- **Security** — private subnet, SG only from app tier

Restore test — production readiness հարց։

---

## 7. CloudWatch և logging

- **Metrics** — CPU, network, custom metrics
- **Alarms** — SNS → email/Slack/PagerDuty
- **Logs** — log groups, retention, **CloudWatch Logs Insights**
- **EventBridge** — schedule / event-driven automation

EC2 agent vs container stdout → CloudWatch Logs։

---

## 8. Cost և Well-Architected (կարճ)

Պիլլարներ՝ Operational Excellence, Security, Reliability, Performance, Cost Optimization, Sustainability։

Practical tips՝

- Right-sizing instances
- Reserved / Savings Plans
- S3 lifecycle
- Idle resources (EIP, unattached EBS)

---

## 9. Troubleshooting AWS scenarios

| Symptoms | Ուղղություն |
|----------|-------------|
| Cannot SSH EC2 | SG (22), NACL, key pair, public IP, subnet route |
| App unreachable from internet | ALB listener, target unhealthy, SG, app bind 0.0.0.0 |
| Private instance no updates | NAT route, NAT GW in public subnet |
| 403 S3 | IAM policy, bucket policy, Block Public Access |
| RDS connection fail | SG, wrong endpoint, subnet group, credentials |

**CLI օրինակներ՝**

```bash
aws sts get-caller-identity
aws ec2 describe-instances --filters "Name=tag:Name,Values=web"
aws elbv2 describe-target-health --target-group-arn ...
aws logs tail /aws/lambda/my-func --follow
```

---

## 10. Advanced topics (overview)

- **ECS/Fargate** — task definition, service, ECR images
- **EKS** — control plane managed, node groups, IRSA (IAM Roles for Service Accounts)
- **Secrets Manager / SSM Parameter Store** — secrets rotation
- **CloudFormation vs Terraform** — both IaC; AWS-native vs multi-cloud
- **Organizations + SCP** — enterprise guardrails

---

## Interview Questions

1. Public vs private subnet — ինչ տարբերություն?
2. Security Group vs NACL?
3. IAM role vs IAM user — երբ որն ես օգտագործում?
4. NAT Gateway vs Internet Gateway?
5. ALB vs NLB — երբ որն?
6. S3-ը ինչպես կապես private EC2-ից առանց public internet-ի? (VPC endpoint)
7. RDS Multi-AZ vs read replica?
8. Auto Scaling Group-ը ինչպես է աշխատում health check-ի հետ?
9. CloudWatch alarm → action flow?
10. How would you troubleshoot «502 from ALB»?
11. What is an IAM policy boundary / SCP (high level)?
12. How do you secure S3 bucket from public exposure?

## Practical Tasks

1. Նկարագրիր (diagram) 3-tier app VPC-ում՝ web (public), app (private), DB (private)։
2. CLI-ով list ար resources մեկ region-ում (instances, buckets)։
3. Ստեղծիր (sandbox) SG rule troubleshooting scenario — fix wrong port।
4. CloudWatch Logs Insights query — top 10 error messages (conceptual)։
5. Կարդա AWS Well-Architected Framework overview և նշիր 3 security best practices։
