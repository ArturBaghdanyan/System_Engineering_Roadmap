# 11 — Ansible

## 1. Ansible-ը ինչ է

**Configuration management** և **automation** գործիք — server-ներ configure անել, deploy, ad-hoc commands, orchestration։

**Կարևոր հատկանիշներ՝**

- **Agentless** — SSH (Linux) / WinRM (Windows), Python on target
- **Declarative-ish** — modules enforce desired state
- **YAML playbooks** — readable
- **Idempotent** — rerun safe (many modules)

```text
Control node (laptop/CI)
    SSH → managed hosts (inventory)
```

---

## 2. Inventory

Static `/etc/ansible/hosts` or `inventory.ini`՝

```ini
[web]
web1.example.com
web2.example.com

[db]
db1.example.com ansible_host=10.0.10.5

[web:vars]
ansible_user=ubuntu
```

**Dynamic inventory** — AWS EC2 plugin, script that outputs JSON。

Groups, vars, `host_vars/`, `group_vars/`։

---

## 3. Ad-hoc commands

```bash
ansible web -m ping
ansible web -a "uptime"
ansible web -b -m apt -a "name=nginx state=present"
```

- `-b` — become (sudo)
- `-m` — module
- `-a` — arguments

---

## 4. Playbook structure

```yaml
---
- name: Configure web servers
  hosts: web
  become: true
  vars:
    app_port: 8080

  tasks:
    - name: Ensure nginx installed
      ansible.builtin.package:
        name: nginx
        state: present

    - name: Deploy config
      ansible.builtin.template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Reload nginx

  handlers:
    - name: Reload nginx
      ansible.builtin.service:
        name: nginx
        state: reloaded
```

**Concepts՝**

- **Task** — one module invocation
- **Handler** — runs once at end if notified
- **Template (Jinja2)** — `{{ variable }}`
- **Tags** — run subset `--tags deploy`

---

## 5. Important modules

| Module | Purpose |
|--------|---------|
| `copy` / `template` | Files |
| `file` | Permissions, symlinks, directories |
| `package` / `apt` / `yum` | Packages |
| `service` / `systemd` | Services |
| `user` / `group` | Accounts |
| `command` / `shell` | Run commands (prefer specialized modules) |
| `uri` | HTTP checks |
| `debug` | Print vars |

Prefer **specialized modules** over raw `shell` — idempotency։

---

## 6. Variables և facts

```yaml
- name: Show hostname
  debug:
    var: ansible_hostname

- name: Gather facts (default)
  setup:
```

Facts — `ansible_distribution`, `ansible_memtotal_mb`, network interfaces。

**Priority** (simplified) — extra vars `-e` > host_vars > group_vars > playbook vars。

---

## 7. Roles

Reusable bundle՝

```text
roles/nginx/
  tasks/main.yml
  handlers/main.yml
  templates/
  defaults/main.yml
  vars/main.yml
```

Playbook-ում՝

```yaml
roles:
  - nginx
  - { role: app, tags: ['deploy'] }
```

**Ansible Galaxy** — community roles (audit before use)։

---

## 8. Vault — secrets

```bash
ansible-vault encrypt group_vars/prod/secrets.yml
ansible-playbook site.yml --ask-vault-pass
```

Encrypted YAML in Git — passwords, API keys。

---

## 9. Ansible vs Terraform

| | Terraform | Ansible |
|---|-----------|---------|
| Focus | Provision infra (create VPC, VM) | Configure OS/app on existing hosts |
| State | State file | No central state (optional AWX) |
| Typical order | Terraform first | Ansible after VM exists |

Միասին՝ **Terraform → Ansible → app deploy**։

---

## 10. AWX / Ansible Automation Platform

Web UI, RBAC, job templates, scheduling, credentials store — enterprise Ansible。

---

## 11. Troubleshooting

- **SSH** — keys, `ansible_user`, bastion (`ProxyJump`)
- **Python** — target needs compatible Python
- **Become** — sudo password, `/etc/sudoers`
- **Check mode** — `ansible-playbook site.yml --check` (dry run)
- **Verbose** — `-vvv`

---

## Interview Questions

1. Agentless — what does it mean?
2. Playbook vs role?
3. Handler vs task?
4. Idempotency — example with `package` module?
5. Inventory static vs dynamic?
6. Ansible vs Terraform?
7. What is Ansible Vault?
8. `--check` mode?
9. When would you use `shell` instead of a module?
10. How do you limit execution to one host or tag?
11. Template vs copy?
12. How would you roll out config change with zero downtime (concept)?

## Practical Tasks

1. Install Ansible; `ansible localhost -m ping -c local` (or WSL target)।
2. Playbook — install nginx, start enabled, deploy static `index.html`।
3. Add handler for config reload।
4. Create role structure for a simple app user + directory।
5. Encrypt a var with `ansible-vault` and run playbook।
