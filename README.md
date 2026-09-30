# System Engineer Interview — ուսումնական նյութեր

Այս repository-ն System Engineer / DevOps հարցազրույցի նախապատրաստման համար է։ Յուրաքանչյուր թեմա ունի իր **`README.md`** ֆայլը՝ հայերեն բացատրություններով, Interview Questions և Practical Tasks բաժիններով։

## Թեմաներ

| # | Թեմա | Նկարագրություն |
|---|------|----------------|
| 01 | [Networking](./01-networking/README.md) | IP, subnet, DNS, TCP/UDP, ports, NAT, SSH, troubleshooting |
| 02 | [Linux](./02-linux/README.md) | Filesystem, processes, systemd, permissions, disk, logs |
| 03 | [Nginx & HTTP](./03-nginx-http/README.md) | Reverse proxy, load balancing, status codes, 502 troubleshooting |
| 04 | [Docker](./04-docker/README.md) | Images, containers, Dockerfile, Compose, volumes, networking |
| 05 | [Production Troubleshooting](./05-production-troubleshooting/README.md) | CPU/RAM/disk, incidents, monitoring vs logging |
| 06 | [Windows Server & SQL](./06-windows-sql/README.md) | Services, AD, PowerShell, SQL հիմունքներ |
| 07 | [Terraform](./07-terraform/README.md) | Infrastructure as Code, state, plan/apply, modules |
| 08 | [AWS Cloud (Advanced)](./08-aws-cloud-advanced/README.md) | VPC, IAM, HA, RDS, S3, CloudWatch, troubleshooting |
| 09 | [Python Automation](./09-python-automation/README.md) | Scripting, boto3, subprocess, logs, DevOps automation |
| 10 | [CI/CD](./10-cicd/README.md) | Pipelines, GitHub Actions, Jenkins, GitOps, deploy strategies |
| 11 | [Ansible](./11-ansible/README.md) | Playbooks, roles, inventory, vault, configuration management |
| 12 | [Kubernetes](./12-kubernetes/README.md) | Pods, Deployments, Services, Ingress, kubectl, troubleshooting |
| 13 | [Monitoring (Grafana & Prometheus)](./13-monitoring-grafana-prometheus/README.md) | Metrics, PromQL, dashboards, alerting, SLO |

## Ինչպես օգտագործել

1. Ընտրիր թեմա և կարդա **`README.md`**-ը ամբողջությամբ։
2. Փորձիր **Practical Tasks** բաժնի command-ները իրական միջավայրում (VM, WSL, Docker Desktop)։
3. Պատասխանիր **Interview Questions**-ին առանց նայելու նյութին, հետո համեմատիր։
4. Troubleshooting թեմաները (01, 05, 03) միացրու մեկ scenario-ի մեջ՝ «կայքը չի բացվում» → DNS → network → Nginx → app → DB։

## Խորհուրդ հարցազրույցի համար

- Մտածիր **բարձրից ներքև**՝ user → DNS → LB/Nginx → application → database → dependencies։
- Միշտ նշիր, **ինչ command** կկիրառես և **ինչ output** կսպասես։
- Production-ում նախ **confirm + scope**, հետո **mitigate**, ապա **root cause** — ոչ թե անմիջապես restart։
