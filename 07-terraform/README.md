# 07 — Terraform

## 1. Infrastructure as Code (IaC)

IaC-ի գաղափարը՝ server-ները, network-ը, database-ը և cloud resource-ները **կոդով** նկարագրելն է, ոչ թե միայն GUI-ով կամ ձեռքով։

```text
.terraform/
main.tf
variables.tf
outputs.tf
     ↓ terraform apply
AWS / Azure / GCP / on-prem resources
```

**Առավելություններ՝**

- Version control (Git) — ով, երբ, ինչ փոխեց
- Repeatability — նույն infra-ն dev/staging/prod-ում
- Documentation — կոդը ինքն infra-ն նկարագրում է
- Automation — CI/CD pipeline-ում apply

---

## 2. Terraform-ը ինչ է

Terraform-ը HashiCorp-ի **declarative** IaC գործիք է։ Դու նկարագրում ես **desired state** («ուզում եմ VPC + 2 EC2»), Terraform-ը հաշվարկում է **diff**-ը և cloud API-ով ստեղծում/թարմացնում/ջնջում է resource-ները։

Terraform-ը **multi-cloud** է՝ մեկ HCL syntax, տարբեր **providers** (AWS, Azure, Google, Kubernetes, Docker և այլն)։

---

## 3. Հիմնական հասկացություններ

### Provider

Provider-ը Terraform-ի և cloud/service API-ի միջև կապն է։

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "eu-central-1"
}
```

### Resource

Resource-ը կոնկրետ managed resource է, որը Terraform-ը կառավարում է։

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0abcdef1234567890"
  instance_type = "t3.micro"
}
```

- `aws_instance` — resource type
- `web` — local name (Terraform-ում reference-ի համար)
- `aws_instance.web` — identifier

### Data source

Արդեն գոյություն ունեցող resource-ից տվյալ կարդալու համար (Terraform-ը չի «ստեղծում» data source-ը)։

```hcl
data "aws_ami" "latest_amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
}
```

---

## 4. HCL — պարզ օրինակ

```hcl
variable "environment" {
  type    = string
  default = "dev"
}

resource "aws_s3_bucket" "logs" {
  bucket = "myapp-logs-${var.environment}"
}

output "bucket_name" {
  value = aws_s3_bucket.logs.bucket
}
```

- **variable** — input
- **output** — apply-ից հետո ցույց տալու արժեք
- **interpolation** — `${var.environment}`

---

## 5. Workflow — init, plan, apply, destroy

```bash
terraform init      # providers/plugins ներբեռնում, backend կարգավորում
terraform fmt       # ֆորմատավորում
terraform validate  # syntax/logic ստուգում
terraform plan      # ինչ կփոխվի (dry-run)
terraform apply     # փոփոխությունները կիրառել
terraform destroy   # resource-ները ջնջել (զգուշությամբ)
```

### Plan vs Apply

- **Plan** — ցույց է տալիս `+ create`, `~ update`, `- destroy`; production-ում plan-ը պահել/review անելը լավ practice է։
- **Apply** — plan-ը հաստատելուց հետո իրական API call-եր։

---

## 6. State file (`terraform.tfstate`)

State-ը Terraform-ի **հիշողությունն** է՝ Terraform resource ID ↔ real cloud ID mapping, dependencies, metadata։

```text
terraform.tfstate  (local, default)
```

**Ինչու է կարևոր**

- Terraform-ը state-ից իմանում է, թե ինչն արդեն ստեղծված է
- State-ը **secret-ներ** կարող է պարունակել → Git-ում commit **չ** անել (`.gitignore`)

### Remote state

Team-ի համար state-ը պահվում է remote backend-ում (S3 + DynamoDB lock, Terraform Cloud, Azure Storage)։

```hcl
terraform {
  backend "s3" {
    bucket         = "company-terraform-state"
    key            = "prod/network/terraform.tfstate"
    region         = "eu-central-1"
    dynamodb_table = "terraform-locks"
  }
}
```

**State locking** — երկու մարդ միաժամանակ apply չանի։

---

## 7. Variables և Outputs

**Variables** — environment-ից, `-var`, `terraform.tfvars`։

```hcl
# terraform.tfvars
environment = "prod"
instance_count = 2
```

```bash
terraform apply -var="environment=staging"
```

**Outputs** — այլ module/system-ին փոխանցելու կամ CI-ում օգտագործելու համար։

```bash
terraform output bucket_name
```

---

## 8. Modules

Module-ը reusable Terraform package է։

```text
modules/
  vpc/
    main.tf
    variables.tf
    outputs.tf
root/
  main.tf   → module "vpc" { source = "./modules/vpc" }
```

Module-ը օգնում է մեծ infra-ն բաժանել logical մասերի և reuse անել։

---

## 9. Lifecycle և drift

### Drift

Եթե ինչ-որ մը AWS console-ից փոխեց resource-ը, Terraform state-ը և reality-ն **չեն համընկնում**։

```bash
terraform plan   # ցույց կտա տարբերությունները
```

### lifecycle block

```hcl
resource "aws_instance" "web" {
  # ...
  lifecycle {
    prevent_destroy = true
    ignore_changes  = [tags]
  }
}
```

---

## 10. Terraform vs Ansible vs Docker

| Գործիք | Ինչ է անում |
|--------|-------------|
| **Terraform** | Infra **provisioning** — VPC, VM, LB, IAM |
| **Ansible** | **Configuration** — package install, config files, service start |
| **Docker** | Application **containerization** — image, run |

Հաճախ միասին՝ Terraform-ը ստեղծում է VM/K8s, Ansible-ը configure է անում, Docker-ը deploy է անում app-ը։

---

## 11. Best practices (interview-ի համար)

- State-ը remote + locked
- `terraform fmt` / `validate` CI-ում
- Plan before apply production-ում
- Secrets — **ոչ** plain text repo-ում; օգտագործել env vars, Vault, AWS Secrets Manager
- `.gitignore` — `*.tfstate`, `*.tfstate.backup`, `.terraform/`
- Naming conventions և tags (owner, environment, cost-center)

---

## 12. Troubleshooting

### Error during apply

1. Կարդալ error message-ը ամբողջությամբ (IAM, quota, invalid AMI)
2. `terraform plan` — ինչ state է սպասում
3. Provider documentation
4. Partial apply — Terraform-ը կարող է retry; state-ում failed resource մնալ

### State corruption / manual fix

- `terraform state list`
- `terraform state show aws_instance.web`
- `terraform import` — արդեն գոյություն ունեցող resource-ը state-ում ավելացնել
- `terraform state rm` — state-ից հանել (cloud-ում resource-ը **չ** է ջնջում)

### Permission errors

- AWS credentials / IAM role
- `provider` block-ի region/account

---

## 13. Պարզ AWS օրինակ (conceptual)

```hcl
terraform {
  required_version = ">= 1.5"
  required_providers {
    aws = { source = "hashicorp/aws", version = "~> 5.0" }
  }
}

provider "aws" {
  region = var.aws_region
}

resource "aws_instance" "app" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"

  tags = {
    Name        = "${var.project}-app"
    Environment = var.environment
  }
}
```

Apply-ից հետո EC2-ն AWS-ում կլինի; `terraform destroy`-ով կջնջվի։

---

## Interview Questions

1. What is Infrastructure as Code?
2. What is Terraform state and why is it important?
3. `terraform plan` vs `terraform apply`?
4. What is a provider?
5. Resource vs data source?
6. Why should you not commit `.tfstate` to Git?
7. What is remote state and state locking?
8. What is drift?
9. What is a Terraform module?
10. Terraform vs Ansible — when would you use each?
11. How would you import an existing resource into Terraform?
12. What happens if two people run `apply` at the same time without locking?

## Practical Tasks

1. Տեղադրիր Terraform և գործարկիր `terraform -version`։
2. Ստեղծիր minimal config (օր. `null_resource` կամ local `docker_container`) և արա `init`, `plan`, `apply`։
3. Փոխիր resource-ի tag/parameter-ը և նորից `plan` — տես diff-ը։
4. Ավելացրիր `variable` և `output`; փոխ_pass ար `terraform.tfvars`-ով։
5. (Եթե cloud access կա) Ստեղծիր S3 bucket կամ small VM, հետո `destroy`։
