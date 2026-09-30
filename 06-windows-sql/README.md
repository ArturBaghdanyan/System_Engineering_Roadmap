# 06 — Windows Server & SQL

# Part 1 — Windows

## 1. Windows Service

Windows Service-ը background process/application է, որը կարող է աշխատել առանց user-ի interactive session-ի։

Services-ը կարող ես կառավարել՝

```text
services.msc
```

կամ PowerShell-ով՝

```powershell
Get-Service
Get-Service -Name Spooler
```

---

## 2. Event Viewer

Event Viewer-ը Windows-ի system/application/security events-ի դիտման գործիք է։

Բացել՝

```text
eventvwr.msc
```

Օգտակար logs՝

- Application
- System
- Security

Եթե service-ը crash է լինում, Event Viewer-ը կարող է օգնել գտնել համապատասխան event-ը։

---

## 3. PowerShell

PowerShell-ը Windows administration-ի scripting և automation shell է։

Օրինակ՝

```powershell
Get-Service
Get-Process
Get-NetTCPConnection
```

---

## 4. Active Directory

Active Directory-ը Microsoft-ի directory service է, որը կազմակերպություններում օգտագործվում է users, computers, groups և policies կառավարելու համար։

Հիմնական հասկացություններ՝

- Domain
- Domain Controller
- User
- Group
- Group Policy
- Organizational Unit

---

## 5. Windows DNS/DHCP

Windows environment-ում DNS-ը name resolution է ապահովում։

DHCP-ը կարող է ավտոմատ տրամադրել՝

- IP
- subnet mask
- gateway
- DNS

---

## 6. Windows troubleshooting

Եթե Windows Server-ը slow է՝ ստուգիր՝

- CPU
- Memory
- Disk
- Processes
- Services
- Event Viewer
- Network
- Recent changes

---

# Part 2 — SQL

## 7. Database

Database-ը structured data պահելու և կառավարելու համակարգ է։

Relational database-ում տվյալները սովորաբար կազմակերպվում են tables-ի մեջ։

---

## 8. SQL

SQL-ը database-ի հետ աշխատելու լեզու է։

SQL-ը database-ը չէ։

---

## 9. SELECT

```sql
SELECT *
FROM users;
```

Բոլոր rows/columns-ը ստանալու օրինակ։

---

## 10. WHERE

```sql
SELECT *
FROM users
WHERE age > 18;
```

Սա վերադարձնում է այն users-ը, որոնց age-ը 18-ից մեծ է։

---

## 11. ORDER BY

```sql
SELECT *
FROM users
ORDER BY age DESC;
```

Sort է անում age-ի նվազման կարգով։

---

## 12. INSERT

```sql
INSERT INTO users (name, age)
VALUES ('Anna', 25);
```

Ավելացնում է row։

---

## 13. UPDATE

```sql
UPDATE users
SET age = 26
WHERE name = 'Anna';
```

Փոխում է matching rows-ը։

Production-ում `UPDATE` առանց `WHERE` անելը վտանգավոր է։

---

## 14. DELETE

```sql
DELETE FROM users
WHERE id = 10;
```

Ջնջում է matching row-երը։

---

## 15. Primary Key

Primary key-ը table-ի row-ը uniquely identify անող key է։

Օրինակ՝

```text
users
id = 1
id = 2
id = 3
```

---

## 16. Foreign Key

Foreign key-ը օգտագործվում է tables-ի միջև relationship ստեղծելու համար։

Օրինակ՝

```text
users
id

orders
user_id
```

`orders.user_id`-ը կարող է reference անել `users.id`-ը։

---

## 17. JOIN

JOIN-ը tables-ից related data միացնելու համար է։

Օրինակ՝

```sql
SELECT users.name, orders.id
FROM users
INNER JOIN orders
ON users.id = orders.user_id;
```

---

## 18. INNER JOIN

Վերադարձնում է այն rows-ը, որոնց համար match կա երկու կողմում։

```text
A ∩ B
```

---

## 19. LEFT JOIN

Ձախ table-ի բոլոր rows-ը պահում է, նույնիսկ եթե աջ table-ում match չկա։

```text
A + matching B
```

---

## 20. Database connectivity troubleshooting

Եթե application-ը չի կարողանում միանալ PostgreSQL-ին՝

Ստուգիր՝

1. Database process-ը running է։
2. Port-ը listening է։
3. Hostname-ը ճիշտ է։
4. Port-ը ճիշտ է։
5. Credentials-ը ճիշտ են։
6. Firewall/network access կա։
7. Database-ը accept է անում remote connections։
8. Application logs-ը։

PostgreSQL default port՝

```text
5432
```

MySQL default port՝

```text
3306
```

---

## Interview Questions

### Windows

1. What is a Windows Service?
2. What is Event Viewer?
3. What is PowerShell?
4. What is Active Directory?
5. What is a Domain Controller?
6. How would you troubleshoot a slow Windows Server?

### SQL

1. What is SQL?
2. What is a database?
3. What is a primary key?
4. What is a foreign key?
5. What is JOIN?
6. INNER JOIN vs LEFT JOIN?
7. What does WHERE do?
8. What does ORDER BY do?
9. What does this query return?

```sql
SELECT *
FROM users
WHERE age > 18;
```

10. How would you troubleshoot an application that cannot connect to PostgreSQL?

## Practical Tasks

### Task 1

Գրիր query, որը վերադարձնում է 18-ից մեծ users։

### Task 2

Գտիր users և orders tables-ը JOIN անելով user name-ը և order ID-ն։

### Task 3

Windows-ում գտիր service-ի status-ը PowerShell-ով։

### Task 4

Event Viewer-ում գտիր Application Error event։
