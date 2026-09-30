# 09 — Python Automation (DevOps / SysAdmin)

## 1. Ինչու Python automation

Python-ը DevOps/System Engineer-ների մոտ տարածված է՝

- Readable syntax
- Rich ecosystem (boto3, requests, paramiko, ansible modules)
- Scripting + small tools + CI steps
- Cross-platform (Linux servers, Windows with care, local laptop)

Օգտագործման դեպքեր՝

- AWS/cloud API calls
- Log parsing, reports
- Deploy/migration scripts
- Health checks, cron jobs
- Glue between tools (Jira, Slack, Git)

---

## 2. Script structure և best practices

```python
#!/usr/bin/env python3
"""Short docstring — what this script does."""

import argparse
import logging
import sys

logging.basicConfig(level=logging.INFO, format="%(levelname)s: %(message)s")
log = logging.getLogger(__name__)


def main() -> int:
    parser = argparse.ArgumentParser(description="Example automation")
    parser.add_argument("--dry-run", action="store_true")
    args = parser.parse_args()
    # ...
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

**Principles՝**

- `if __name__ == "__main__"` — importable modules
- **Exit codes** — 0 success, non-zero failure (CI-ում կարևոր)
- **Logging**, ոչ print-only production scripts-ում
- **argparse** / click — CLI flags
- **Type hints** — maintainability
- **Virtual env** — `python -m venv .venv`, `requirements.txt` or `pyproject.toml`

---

## 3. Files, paths, subprocess

```python
from pathlib import Path

config = Path("/etc/app/config.yaml")
if not config.exists():
    raise FileNotFoundError(config)

text = config.read_text(encoding="utf-8")
```

**Subprocess** (shell injection-ից խուսափել)՝

```python
import subprocess

result = subprocess.run(
    ["systemctl", "is-active", "nginx"],
    capture_output=True,
    text=True,
    check=False,
)
if result.returncode != 0:
    log.error("nginx not active: %s", result.stderr)
```

Մի օգտագործիր `shell=True` user input-ի հետ։

---

## 4. HTTP automation (requests)

## 4. Ամբողջական օրինակ՝ HTTP health check

Հետևյալ script-ը ստանում է մեկ կամ մի քանի URL, ստուգում է դրանց հասանելիությունը և ձախողման դեպքում վերադարձնում է ոչ զրոյական exit code։ Այն օգտագործում է Python-ի ստանդարտ գրադարանը, այսինքն՝ լրացուցիչ package տեղադրել պետք չէ։ Պահպանիր այն, օրինակ, `check_urls.py` անունով։

```python
#!/usr/bin/env python3
"""Check HTTP endpoints and return a failure code if any endpoint is unhealthy."""

import argparse
import logging
import sys
from urllib.error import HTTPError, URLError
from urllib.request import urlopen

logging.basicConfig(level=logging.INFO, format="%(levelname)s: %(message)s")
log = logging.getLogger(__name__)


def check_url(url: str, timeout: float) -> bool:
    try:
        with urlopen(url, timeout=timeout) as response:
            if 200 <= response.status < 400:
                log.info("%s: HTTP %s", url, response.status)
                return True
            log.error("%s: unexpected HTTP status %s", url, response.status)
    except HTTPError as error:
        log.error("%s: HTTP %s", url, error.code)
    except (URLError, TimeoutError) as error:
        log.error("%s: connection failed: %s", url, error)
    return False


def main() -> int:
    parser = argparse.ArgumentParser(description="Check HTTP endpoint health")
    parser.add_argument("urls", nargs="+", help="one or more HTTP or HTTPS URLs")
    parser.add_argument("--timeout", type=float, default=5.0, help="timeout in seconds")
    args = parser.parse_args()
    if args.timeout <= 0:
        parser.error("--timeout must be greater than zero")

    failed = False
    for url in args.urls:
        if not check_url(url, args.timeout):
            failed = True
    return 1 if failed else 0


if __name__ == "__main__":
    sys.exit(main())
```

Գործարկում՝

```bash
python check_urls.py --timeout 3 https://example.com https://example.com/health
echo $?
```

Windows PowerShell-ում exit code-ը դիտելու համար օգտագործիր `$LASTEXITCODE`։ `200–399` կարգավիճակները համարվում են հասանելի, իսկ `4xx/5xx`, DNS/կապի սխալը կամ timeout-ը՝ ձախողում։ Բոլոր URL-ները ստուգվում են, ոչ միայն առաջին անհաջողը։ CI pipeline-ը կամ cron-ը կարող է exit code `0`-ով շարունակել, իսկ `1`-ի դեպքում նշել job-ը որպես ձախողված։ Timeout-ը սահմանափակում է մեկ հարցման սպասումը, որպեսզի անհասանելի host-ը անվերջ չկախի script-ը։

Իրական միջավայրում URL-ները սովորաբար վերցնում են argument-ից, ֆայլից կամ environment-ից։ Գաղտնի token-ները մի՛ տեղադրիր URL-ի մեջ, քանի որ URL-ը կարող է հայտնվել log-երում։ Բազմաթիվ և ժամանակավոր ցանցային խափանումների համար ավելացրու սահմանափակ retry/backoff՝ ուշացումն ու server-ին ավելորդ հարցումները վերահսկելու համար։

---

## 5. AWS automation (boto3)

Credentials՝ environment, `~/.aws/credentials`, **IAM role on EC2/Lambda** (preferred)։

```python
import boto3

ec2 = boto3.client("ec2", region_name="eu-central-1")
instances = ec2.describe_instances(
    Filters=[{"Name": "instance-state-name", "Values": ["running"]}]
)
for r in instances["Reservations"]:
    for i in r["Instances"]:
        print(i["InstanceId"], i.get("Tags"))
```

Paginators — large lists (S3 objects, log streams)։

---

## 6. SSH / remote hosts (paramiko)

```python
import paramiko

client = paramiko.SSHClient()
client.set_missing_host_key_policy(paramiko.AutoAddPolicy())  # prod: known_hosts
client.connect("host.example.com", username="deploy", key_filename="/path/key")
stdin, stdout, stderr = client.exec_command("uptime")
print(stdout.read().decode())
client.close()
```

Production-ում՝ keys from vault, jump host, logging, idempotent operations։

---

## 7. Parsing logs և data

```python
import re
from collections import Counter

pattern = re.compile(r"ERROR .+")
counts = Counter()
with open("/var/log/app/app.log", encoding="utf-8", errors="replace") as f:
    for line in f:
        if pattern.search(line):
            counts[line.strip()] += 1
```

JSON logs — `json.loads(line)` per line (structured logging)։

---

## 8. Configuration

- **Environment variables** — `os.environ["DB_HOST"]`
- **YAML/JSON** — `import yaml` / `json.load`
- **Secrets** — never hardcode; AWS Secrets Manager, vault, CI secrets

```python
import os

db_host = os.environ.get("DB_HOST", "localhost")
```

---

## 9. Testing automation scripts

- **pytest** — unit tests for pure functions
- Mock external APIs (`unittest.mock`, `moto` for AWS)
- Dry-run mode for destructive operations

---

## 10. Packaging և deployment

- Pin dependencies — `requirements.txt`
- Run in CI — lint (ruff/flake8), format (black), tests
- Schedule — cron, systemd timer, Kubernetes CronJob, Lambda EventBridge

---

## Interview Questions

1. Why use Python for DevOps automation vs Bash only?
2. How do you pass secrets to a Python script safely?
3. `subprocess.run` vs `os.system`?
4. How would you call AWS APIs from Python?
5. Exit code importance in CI/cron?
6. How do you make a script idempotent?
7. Virtual environment — why?
8. How would you parse a 10GB log file efficiently?
9. Error handling strategy for a deploy script?
10. How do you test boto3 code without hitting real AWS?

## Practical Tasks

1. Script — read list of URLs from file, HTTP GET, print status code summary।
2. Script — list running EC2 instances (tag Name) with boto3 (sandbox account)।
3. Add `--dry-run` and logging to an existing one-liner script।
4. Parse nginx access log — top 5 IPs by request count।
5. Wrap a shell command with subprocess and map failures to exit code 1।
