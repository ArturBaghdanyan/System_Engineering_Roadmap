# 01 — Networking

## 1. Network-ի հիմնական գաղափարը

Network-ը սարքերի միջև կապի համակարգ է։ Client-ը request է ուղարկում server-ին, իսկ server-ը response է վերադարձնում։

```text
Client → Network → Server
```

Հիմնական հասկացություններն են՝ IP, subnet, gateway, DNS, ports, TCP/UDP, NAT։

---

## 2. IP Address

IP address-ը ցանցում սարքը կամ interface-ը նույնականացնող հասցե է։

IPv4 օրինակ՝

```text
192.168.1.10
```

IPv4-ը 32-bit է և սովորաբար գրվում է 4 octet-ով։

IPv6-ը 128-bit է։

```text
2001:db8::1
```

### Private IP

Private IP-ները օգտագործվում են ներքին ցանցերում և ուղղակիորեն Internet-ում routable չեն։

Հիմնական ranges՝

```text
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```

### Public IP

Public IP-ն Internet-ում routable հասցե է։ Տնային router-ը հաճախ ունի public IP Internet-ի կողմում և private IP-ներ LAN-ի ներսում։

---

## 3. Subnet

Subnet-ը network-ը փոքր logical ցանցերի բաժանելու միջոց է։

Օրինակ՝

```text
192.168.1.0/24
```

`/24` նշանակում է, որ առաջին 24 bit-ը network part-ն է։

---

## 4. Default Gateway

Gateway-ը այն սարքն է, որին host-ը դիմում է այլ network գնալու համար։

Օրինակ՝

```text
PC 192.168.1.20
     ↓
Gateway 192.168.1.1
     ↓
Internet
```

---

## 5. DNS

DNS-ը domain name-ը IP address-ի հետ resolve անող համակարգ է։

```text
example.com
     ↓ DNS
93.184.216.34
```

Հիմնական record-ներ՝

- `A` — domain → IPv4
- `AAAA` — domain → IPv6
- `CNAME` — alias դեպի այլ hostname
- `MX` — mail server
- `TXT` — text/verification տվյալներ

Useful commands:

```bash
nslookup example.com
dig example.com
```

---

## 6. DHCP

DHCP-ը client-ին ավտոմատ network configuration է տալիս՝ IP, subnet mask, gateway, DNS և այլն։

---

## 7. TCP vs UDP

### TCP

- Connection-oriented
- Reliable
- Packets-ի order-ը վերահսկվում է
- Retransmission ունի
- Օգտագործվում է, երբ տվյալների կորուստը խնդիր է

### UDP

- Connectionless
- Ավելի քիչ overhead
- Delivery guarantee չունի
- Հարմար է latency-sensitive traffic-ի համար

Օրինակներ՝ DNS-ը հաճախ UDP 53, իսկ HTTP/HTTPS-ը սովորաբար TCP-ի վրա են աշխատում։

---

## 8. Ports

Port-ը օգնում է հասկանալ՝ տվյալ host-ի որ service-ին պետք է ուղարկել traffic-ը։

Հաճախ հանդիպող ports՝

| Port | Service |
|---|---|
| 22 | SSH |
| 53 | DNS |
| 80 | HTTP |
| 443 | HTTPS |
| 25 | SMTP |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 6379 | Redis |

---

## 9. HTTP vs HTTPS

HTTP-ը application-layer protocol է web communication-ի համար։

HTTPS-ը HTTP + TLS է։

```text
HTTP  → usually 80
HTTPS → usually 443
```

HTTPS-ը ապահովում է encryption և server authentication TLS-ի միջոցով։

---

## 10. NAT

NAT-ը network address translation է։
տեխնոլոգիա է, որը թույլ է տալիս տեղային (Private/Local) IP հասցեները թարգմանել հանրային (Public) IP հասցեների, 
երբ տվյալները դուրս են գալիս դեպի Ինտերնետ:

Այն սովորաբար աշխատում է երթուղիչների (Router) կամ Firewall-ների մակարդակում:

Տիպիկ տնային network՝

```text
Private PC
192.168.1.10
      ↓
Router / NAT
      ↓
Public IP
      ↓
Internet
```

Մի public IP կարող է սպասարկել բազմաթիվ private hosts՝ տարբեր source ports-երի միջոցով։

---

## 11. SSH

SSH-ը remote server-ին secure connection ստեղծելու protocol է։

```bash
ssh user@192.168.1.10
```

Default port՝ `22`։

---

## 12. Useful troubleshooting commands

### ping

```bash
ping 8.8.8.8
```

Ստուգում է IP-level reachability-ի հիմնական ասպեկտը։

### curl

```bash
curl -I https://example.com
```

Օգտակար է HTTP/HTTPS response-ը ստուգելու համար։

### traceroute

Linux:

```bash
traceroute example.com
```

Windows:

```cmd
tracert example.com
```

Օգնում է տեսնել route-ի hops-երը։

### ss

```bash
ss -tulpn
```

Ցույց է տալիս listening sockets/connections-ի տվյալներ։

---

## 13. Troubleshooting օրինակ

Եթե `example.com`-ը չի բացվում՝

1. Ստուգել network connectivity։
2. `ping` կամ այլ reachability test անել։
3. Ստուգել DNS-ը `nslookup`/`dig`-ով։
4. Ստուգել port-ը։
5. `curl`-ով ստուգել HTTP response-ը։
6. Եթե server-ի access կա՝ ստուգել service և logs։

Եթե IP-ով աշխատում է, բայց domain-ով՝ ոչ, DNS-ը հիմնական կասկածներից է։

---

## Interview Questions

1. What is DNS?
2. TCP vs UDP?
3. Public vs private IP?
4. What is NAT?
5. What is a subnet?
6. What is a default gateway?
7. What is port 22?
8. What is port 443?
9. What does `curl` do?
10. What does `ss -tulpn` show?
11. IP-ով հասանելի է, domain-ով՝ ոչ. ի՞նչ կստուգես։
12. Server-ը ping-ին պատասխանում է, բայց website-ը չի բացվում. ի՞նչ կստուգես։

## Practical Tasks

```bash
ping 8.8.8.8
nslookup google.com
curl -I https://google.com
ss -tulpn
```

Յուրաքանչյուր command-ի output-ը փորձիր բացատրել։
