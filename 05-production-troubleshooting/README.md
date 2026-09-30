# 05 — Production Troubleshooting

## 1. Գլխավոր սկզբունքը

Production troubleshooting-ի նպատակը միայն «restart անել»-ը չէ։

Պետք է գտնել root cause-ը կամ նվազագույնը արագ մեկուսացնել խնդրի layer-ը։

```text
Problem
  ↓
Confirm
  ↓
Scope
  ↓
Logs / Metrics
  ↓
Resources
  ↓
Network
  ↓
Recent changes
  ↓
Fix / Rollback
  ↓
Verify
  ↓
Document
```

---

## 2. Առաջին հարցերը

Երբ production incident է լինում՝

- Ի՞նչն է չի աշխատում։
- Բոլո՞ր user-ներն են affected։
- Ե՞րբ սկսվեց։
- Վերջերս deployment եղե՞լ է։
- Error code-ը ո՞րն է։
- Կարո՞ղ ենք reproduce անել։
- Service-ը running է՞։

---

## 3. CPU 100%

Սկզբում՝

```bash
top
```

կամ

```bash
htop
```

Process list՝

```bash
ps aux --sort=-%cpu | head
```

Հետո պարզիր՝ ինչ է անում process-ը։

Check՝

- logs
- recent deployment
- infinite loop
- unusual traffic
- background job
- runaway process

Միանգամից process kill անել պետք չէ, հատկապես production-ում։

---

## 4. Memory-ը անընդհատ աճում է

Check՝

```bash
free -h
top
ps aux --sort=-%mem | head
```

Հնարավոր պատճառներ՝

- memory leak
- cache growth
- too many processes
- traffic increase
- configuration problem

Application logs և metrics-ը նույնպես պետք է ստուգել։

---

## 5. Disk 95%+

```bash
df -h
```

Հետո գտնել մեծ directories՝

```bash
du -xh /var | sort -h
```

Մեծ files՝

```bash
find /var -type f -size +500M
```

Հաճախ պատճառ կարող են լինել՝

- logs
- backups
- temporary files
- database files
- Docker images/volumes

Production-ում ֆայլեր ջնջելուց առաջ հասկանալ՝ ինչ են և ինչ retention policy կա։

---

## 6. Service Down

Օրինակ՝ Nginx։

```bash
systemctl status nginx
```

Logs՝

```bash
journalctl -u nginx -n 100
```

Config՝

```bash
nginx -t
```

Ports՝

```bash
ss -tulpn
```

---

## 7. Website Down

Համակարգված մոտեցում՝

```text
DNS
 ↓
Network
 ↓
Port
 ↓
Nginx / Load Balancer
 ↓
Application
 ↓
Database / Dependencies
```

Օրինակ՝

```bash
nslookup example.com
curl -I https://example.com
ss -tulpn
systemctl status nginx
```

---

## 8. 502 Bad Gateway

Typical chain՝

```text
Client
 ↓
Nginx
 ↓
Backend
```

Եթե Nginx-ը տալիս է 502՝ ստուգիր backend-ը։

```bash
curl http://127.0.0.1:3000
```

Հետո՝

- application status
- application logs
- port
- Nginx config
- network/firewall

---

## 9. Deployment-ից հետո error

Եթե deployment-ից անմիջապես հետո error սկսվեց՝ recent change-ը կարևոր clue է։

Ստուգիր՝

1. Deployment version։
2. Application logs։
3. Configuration/environment variables։
4. Dependencies։
5. Database migrations։
6. Resource usage։
7. Health checks։

Եթե rollback-ը նախատեսված/անվտանգ է և incident-ը պահանջում է՝ rollback կարող է լինել mitigation։

---

## 10. "Works on my machine"

Համեմատիր environments-ը։

- OS
- runtime version
- dependencies
- environment variables
- network access
- database
- filesystem
- permissions
- configuration
- container image/version

Հաճախ խնդիրը environment difference-ն է։

---

## 11. Monitoring vs Logging

### Monitoring

Ցույց է տալիս համակարգի վիճակի metrics։

Օրինակ՝

- CPU
- RAM
- disk
- latency
- request rate
- error rate

### Logging

Գրանցում է իրադարձություններ և application/system messages։

Monitoring-ը հաճախ ասում է՝ **ինչ-որ բան վատ է**։

Logs-ը օգնում են հասկանալ՝ **ինչ է տեղի ունեցել**։

---

## 12. Incident response

Լավ մոտեցում՝

```text
Detect
→ Confirm
→ Assess impact
→ Mitigate
→ Investigate
→ Resolve
→ Verify
→ Document
```

---

## Interview Questions

1. CPU 100% — what do you do?
2. Disk 95% — what do you do?
3. RAM continuously grows — what do you investigate?
4. Website is down — how do you troubleshoot?
5. What is 502?
6. Deployment broke production — what do you do?
7. What is rollback?
8. Monitoring vs logging?
9. "Works on my machine" — what do you check?
10. Would you restart production immediately? Why/why not?

## Practical Scenarios

### Scenario A

```text
Nginx = running
Backend = stopped
Browser = 502
```

Գտիր root cause-ի chain-ը։

### Scenario B

```text
Disk = 98%
/var/log = 200 GB
```

Բացատրիր՝ ինչ կանես անվտանգ ձևով։

### Scenario C

```text
CPU = 100%
process = node
```

Գրիր troubleshooting plan։
