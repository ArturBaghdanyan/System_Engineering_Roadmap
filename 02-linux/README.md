# 02 — Linux

## 1. Linux-ը որպես System Engineer-ի գործիք

System Engineer-ը Linux-ում հաճախ աշխատում է command line-ով՝

- process-ներ
- services
- users
- permissions
- disk
- memory
- network
- logs

---

## 2. Filesystem

Հիմնական directories՝

```text
/
├── etc
├── var
├── home
├── tmp
├── usr
└── opt
```

### `/etc`

Configuration files։

### `/var/log`

Logs։

### `/home`

Users-ի home directories։

---

## 3. Basic commands

```bash
pwd
ls
cd
mkdir
touch
cp
mv
rm
cat
less
head
tail
```

Օրինակ՝

```bash
tail -n 100 app.log
```

Վերջին 100 տողը։

Live logs՝

```bash
tail -f app.log
```

---

## 4. grep

Տեքստ որոնելու համար։

```bash
grep "ERROR" app.log
```

Case-insensitive՝

```bash
grep -i "error" app.log
```

---

## 5. find

Ֆայլեր որոնելու համար։

```bash
find /var/log -name "*.log"
```

Մեծ ֆայլերի որոնման օրինակ՝

```bash
find /var -type f -size +500M
```

---

## 6. Processes

Process-ը running program-ի instance է։

```bash
ps aux
```

Interactive monitoring՝

```bash
top
htop
```

CPU-heavy process գտնելու համար կարող ես օգտագործել `top` կամ `ps`։

---

## 7. kill

Graceful termination-ի համար՝

```bash
kill PID
```

Force kill՝

```bash
kill -9 PID
```

`kill -9`-ը պետք չէ օգտագործել որպես առաջին քայլ։ Սովորաբար նախ փորձում ենք սովորական termination։

---

## 8. systemd և systemctl

Շատ Linux distribution-ներում systemd-ն service manager է։

Status՝

```bash
systemctl status nginx
```

Start՝

```bash
sudo systemctl start nginx
```

Stop՝

```bash
sudo systemctl stop nginx
```

Restart՝

```bash
sudo systemctl restart nginx
```

Reload՝

```bash
sudo systemctl reload nginx
```

Enable at boot՝

```bash
sudo systemctl enable nginx
```

---

## 9. journalctl

Service logs դիտելու համար՝

```bash
journalctl -u nginx
```

Վերջին logs-ը՝

```bash
journalctl -u nginx -n 100
```

Live՝

```bash
journalctl -u nginx -f
```

---

## 10. Permissions

Linux permissions-ը սովորաբար բաժանվում է՝

```text
owner
group
others
```

Օրինակ՝

```text
rwxr-xr-x
```

նշանակում է՝

```text
owner  = rwx
group  = r-x
others = r-x
```

Թվային ձևը՝

```text
r = 4
w = 2
x = 1
```

Ուստի՝

```text
7 = 4+2+1 = rwx
5 = 4+1   = r-x
```

`chmod 755 script.sh` նշանակում է՝ owner-ը read/write/execute, իսկ group/others-ը read/execute։

---

## 11. Disk

### df

Filesystem-ի օգտագործումը՝

```bash
df -h
```

### du

Directory/file-ի իրական օգտագործումը՝

```bash
du -sh /var/log
```

Օրինակ՝ եթե disk-ը 95% է՝

```bash
df -h
du -xh /var | sort -h
```

Զգուշությամբ աշխատիր production-ում։

---

## 12. Memory

```bash
free -h
```

CPU/load-ի ընդհանուր պատկեր՝

```bash
uptime
```

---

## 13. Network

Listening ports՝

```bash
ss -tulpn
```

HTTP test՝

```bash
curl -I http://localhost
```

---

## 14. Users

```bash
whoami
id
```

User-ի home/config տվյալների հիմնական աղբյուրներից է `/etc/passwd`։

`sudo`-ն թույլ է տալիս authorized user-ին կատարել privileged command։

---

## 15. Environment Variables

Տեսնել՝

```bash
printenv
```

Կոնկրետ variable՝

```bash
echo $PATH
```

Environment variable-ները հաճախ օգտագործվում են application configuration-ի համար։

---

## 16. Troubleshooting framework

Երբ service-ը չի աշխատում՝

```text
1. Confirm the problem
2. Check service status
3. Check logs
4. Check process
5. Check ports
6. Check CPU/RAM/Disk
7. Check network
8. Check recent changes
9. Fix or rollback
10. Verify
```

Օրինակ՝

```bash
systemctl status nginx
journalctl -u nginx -n 100
ss -tulpn
df -h
free -h
```

---

## Interview Questions

1. Process vs thread?
2. What does `ps aux` show?
3. `kill` vs `kill -9`?
4. What is systemd?
5. What does `systemctl status` do?
6. What is `journalctl`?
7. `df` vs `du`?
8. Explain `chmod 755`.
9. How do you find large files?
10. How do you check RAM?
11. How do you check listening ports?
12. How do you troubleshoot a failed service?

## Practical Tasks

1. Գտիր `/var/log`-ի մեծ directories-ը։
2. Գտիր CPU-heavy process-ը։
3. Ստուգիր nginx-ի status-ը։
4. Դիտիր nginx-ի վերջին 100 log line-ը։
5. Գտիր listening ports-ը։
