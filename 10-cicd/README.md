# 10 — CI/CD Tools

## 1. CI vs CD

| Տerm                            | Նշանակություն                                 |
| ------------------------------- | --------------------------------------------- |
| **CI** (Continuous Integration) | Code merge → build → test ավտոմատ             |
| **CD** (Continuous Delivery)    | Artifact deploy-ready, manual approve to prod |
| **CD** (Continuous Deployment)  | Prod deploy ավտոմատ after tests               |

```text
git push → pipeline → build → test → scan → artifact → deploy staging → (approve) → deploy prod
```

---

## 2. Pipeline stages (typical)

1. **Checkout** — source code
2. **Lint / format** — code quality
3. **Unit tests**
4. **Build** — binary, Docker image, package
5. **Integration tests** — DB, API mocks
6. **Security** — SAST, dependency scan, container scan
7. **Publish** — registry (ECR, Docker Hub), S3, Maven
8. **Deploy** — Terraform, Ansible, kubectl, SSH

Fail fast — early stage errors չեն հասնում production։

---

## 3. Popular tools

| Tool                  | Ծանոթություն                             |
| --------------------- | ---------------------------------------- |
| **GitHub Actions**    | YAML workflows, marketplace actions      |
| **GitLab CI**         | `.gitlab-ci.yml`, integrated with GitLab |
| **Jenkins**           | Plugins, self-hosted, Groovy/Jenkinsfile |
| **Azure DevOps**      | Pipelines, boards, artifacts             |
| **CircleCI / Travis** | Cloud CI                                 |
| **Argo CD / Flux**    | GitOps for Kubernetes                    |

Interview-ում հ часто՝ «ինչ tool եք օգտագործել» + **concepts** (stages, artifacts, secrets)։

---

## 4. GitHub Actions (conceptual)

```yaml
name: Python CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read

jobs:
  test:
    name: Run unit tests
    runs-on: ubuntu-latest
    steps:
      - name: Get repository files
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install test dependency
        run: python -m pip install pytest

      - name: Run tests
        run: python -m pytest -q
```

Այս workflow-ը պահվում է repository-ի `.github/workflows/ci.yml` ֆայլում։ `on`-ը որոշում է՝ երբ սկսել pipeline-ը. այստեղ այն աշխատում է, երբ փոփոխություն է push արվում `main` ճյուղ, և երբ pull request-ի թիրախը `main` է։ Pull request-ի ստուգումը թույլ է տալիս սխալը գտնել մինչև փոփոխությունը միացվի հիմնական ճյուղին։

GitHub-ը ստեղծում է ժամանակավոր `ubuntu-latest` runner՝ վիրտուալ մեքենա, որտեղ կատարվում են job-ի քայլերը։ `steps`-ը հերթականությամբ վերցնում է repository-ի ֆայլերը, տեղադրում Python-ը, տեղադրում pytest-ը և գործարկում թեստերը։ `uses`-ը կանչում է վերաօգտագործվող Action, իսկ `run`-ը կատարում է shell հրաման։ Եթե որևէ հրաման ավարտվի ոչ զրոյական exit code-ով, քայլը և job-ը ձախողվում են. pull request-ում դա երևում է որպես չանցած ստուգում։

Օրինակը ենթադրում է, որ repository-ում կան pytest թեստեր։ Իրական հավելվածում test dependency-ից առաջ տեղադրիր նաև հավելվածի dependencies-ը (օրինակ՝ `requirements.txt`-ից), իսկ թեստերը պահիր `tests/` պանակում։ `permissions: contents: read`-ը workflow token-ին տալիս է միայն repository-ն կարդալու նվազագույն թույլտվությունը. deploy job-ի համար ավելացրու միայն անհրաժեշտ permissions-ը։

GitHub Actions secret-ները հասանելի են `${{ secrets.SECRET_NAME }}` արտահայտությամբ, բայց դրանց արժեքները չպետք է տպվեն log-ում։ AWS deploy-ի համար նախընտրիր OIDC role assumption-ը՝ երկարաժամկետ access key պահելու փոխարեն. այդ դեպքում պետք են համապատասխան `id-token: write` permission և AWS-ի trust policy։ Թեստերի այս job-ին գաղտնիքներ պետք չեն։

---

## 5. Jenkins (conceptual)

- **Controller + agents** (executors)
- **Pipeline** — Declarative (`Jenkinsfile`) or Scripted
- **Plugins** — Git, Docker, Kubernetes, credentials binding

```groovy
pipeline {
  agent any
  stages {
    stage('Build') { steps { sh 'make build' } }
    stage('Test')  { steps { sh 'make test' } }
  }
}
```

---

## 6. Artifacts և caching

- **Artifact** — reproducible output (jar, `app.tar.gz`, Docker image digest)
- **Immutable tags** — prod deploy by digest, not `:latest`
- **Cache** — dependencies (npm, pip, Docker layers) — faster pipelines

---

## 7. Deployment strategies

| Strategy       | Idea                           |
| -------------- | ------------------------------ |
| **Rolling**    | Replace instances gradually    |
| **Blue/Green** | Two envs, switch traffic       |
| **Canary**     | Small % traffic to new version |
| **Recreate**   | Stop all, start new (downtime) |

Kubernetes + ALB/Ingress — natural fit for rolling/canary (with tools)։

---

## 8. GitOps

Desired state in **Git** (manifests, Helm, Kustomize) → controller (Argo CD) syncs cluster։

```text
Developer PR → merge → Argo detects drift → apply to cluster
```

Benefits — audit trail, rollback = revert commit。

---

## 9. Secrets, environments, approvals

- **Environments** — dev / staging / prod protection rules
- **Manual approval** — prod gate
- **Secrets rotation** — vault, cloud secret manager
- **Least privilege** — deploy role only what needed

---

## 10. Troubleshooting failed pipelines

1. Read **failed stage log** (first error, not last)
2. Reproduce locally — same Docker image / same command
3. Flaky tests — retry vs fix
4. Resource limits — OOM on runner, timeout
5. Credential expiry — OIDC trust, token TTL

---

## Interview Questions

1. CI vs CD — difference?
2. What stages would you put in a web app pipeline?
3. How do you store secrets in CI?
4. Blue/green vs rolling deploy?
5. What is GitOps?
6. Docker image `:latest` in prod — good or bad? Why?
7. How would you rollback a bad deploy?
8. Jenkins vs GitHub Actions — tradeoffs?
9. What is an artifact and why pin versions?
10. How do you prevent deploying untested code to production?
11. OIDC federation to AWS from GitHub — benefit?
12. What is a flaky test and how do you handle it?

## Practical Tasks

1. Write minimal GitHub Actions workflow — pytest on push।
2. Draw pipeline diagram for Docker app → ECR → ECS/EKS deploy।
3. List 5 checks you'd add before prod deploy।
4. Explain how you'd implement manual approval for production।
5. Compare recreate vs rolling for a stateful app (discussion)।
