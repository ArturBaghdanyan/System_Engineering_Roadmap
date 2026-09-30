# 03 — Nginx & HTTP

## 1. What is Nginx?

Nginx-ը web server և reverse proxy է։

Կարող է կատարել՝

- serve static files
- reverse proxy
- load balancing
- TLS termination
- routing
- caching

Typical architecture՝

```text
Client
   ↓
Nginx :443
   ↓
Application :3000
```

---

## 2. Reverse Proxy

Client-ը ուղղակիորեն չի խոսում backend-ի հետ։

```text
Client → Nginx → Node.js
```

Nginx-ը ստանում է request-ը և փոխանցում upstream application-ին։

---

## 3. Load Balancing

Մի քանի backend server-ի դեպքում՝

```text
             ┌→ App 1
Client → Nginx├→ App 2
             └→ App 3
```

Nginx-ը կարող է request-ները բաշխել backend-ների միջև։

---

## 4. HTTP status codes

### 2xx

Success։

`200 OK`

### 3xx

Redirection։

`301`, `302`

### 4xx

Client/request-side issue։

`400`, `401`, `403`, `404`

### 5xx

Server-side կամ upstream-related issue։

`500`, `502`, `503`, `504`

---

## 5. 500 vs 502 vs 503 vs 504

### 500 Internal Server Error

Application-ը server-side error է վերադարձրել։

### 502 Bad Gateway

Reverse proxy/gateway-ը upstream-ից valid response չի ստացել։

Օրինակ՝

```text
Nginx → Node.js :3000 ❌
```

### 503 Service Unavailable

Service-ը հասանելի չէ կամ temporarily unavailable է։

### 504 Gateway Timeout

Gateway/proxy-ն սպասել է upstream response-ի, բայց timeout է եղել։

---

## 6. Nginx config test

Փոփոխությունից առաջ՝

```bash
sudo nginx -t
```

Եթե syntax-ը ճիշտ է՝ reload։

```bash
sudo systemctl reload nginx
```

Reload-ը սովորաբար նախընտրելի է, քան անտեղի restart-ը։

---

## 7. Logs

Common locations՝

```text
/var/log/nginx/access.log
/var/log/nginx/error.log
```

Դիտել՝

```bash
tail -f /var/log/nginx/error.log
```

---

## 8. Example reverse proxy

Conceptual config՝

```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Այս դեպքում Nginx-ը request-ը փոխանցում է local port `3000`։

**Headers** — backend-ը հաճախ պետք է իմանա իրական client IP-ն և scheme-ը (HTTP/HTTPS), հատկապես TLS termination Nginx-ում լինելու դեպքում։

---

## 9. Upstream block և load balancing

```nginx
upstream backend {
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
    server 127.0.0.1:3003;
}

server {
    listen 80;

    location / {
        proxy_pass http://backend;
    }
}
```

Default algorithm-ը **round-robin** է։ Interview-ում կարող են հարցնել նաև `ip_hash` (session stickiness) կամ `least_conn`։

---

## 10. Static files vs proxy

```nginx
location /static/ {
    alias /var/www/app/static/;
}

location / {
    proxy_pass http://127.0.0.1:3000;
}
```

Nginx-ը static content-ը կարող է serve անել **ուղիղ** (արագ, քիչ load backend-ի վրա), իսկ dynamic request-ները proxy անել։

---

## 11. HTTPS / TLS termination

```text
Client --HTTPS--> Nginx :443 --HTTP--> Backend :3000
```

Nginx-ը certificate-ը պահում է (`ssl_certificate`, `ssl_certificate_key`), backend-ը հաճախ HTTP-ով աշխատում է ներքին ցանցում։

Interview-ում՝ **TLS termination** = encryption-ը ավարտվում է Nginx-ում։

---

## 12. Useful directives

| Directive | Նշանակություն |
|-----------|----------------|
| `client_max_body_size` | Upload file-ի max size |
| `proxy_read_timeout` | Upstream-ից response սպասելու timeout |
| `proxy_connect_timeout` | Upstream-ին միանալու timeout |
| `access_log` / `error_log` | Log path և level |

Timeout-ները կապված են **504 Gateway Timeout** error-ի հետ։

---

## 13. 502 troubleshooting

Եթե browser-ը ստանում է 502՝

1. Ստուգիր Nginx status-ը։
2. `nginx -t`
3. Կարդա error log-ը։
4. Ստուգիր upstream application-ը։
5. Ստուգիր՝ port-ը listening է։
6. Test արա backend-ը direct՝

```bash
curl http://127.0.0.1:3000
```

7. Ստուգիր application logs-ը։
8. Ստուգիր firewall/network/config։

---

## 14. HTTP request-ի ընդհանուր ընթացքը

```text
Browser
 ↓
DNS
 ↓
IP
 ↓
TCP connection
 ↓
TLS (HTTPS)
 ↓
HTTP request
 ↓
Nginx
 ↓
Backend
 ↓
Response
```

---

## Interview Questions

1. What is Nginx?
2. What is a reverse proxy?
3. Reverse proxy vs load balancer?
4. What is HTTP 502?
5. 500 vs 502?
6. What is 504?
7. How do you test Nginx config?
8. Where are Nginx logs?
9. How do you reload Nginx?
10. How would you troubleshoot 502?

## Practical Tasks

1. Տեղադրիր Nginx։
2. Ստուգիր `nginx -t`։
3. Գտիր access/error logs-ը։
4. Ստեղծիր reverse proxy դեպի local application։
5. Դիտավորյալ սխալ upstream port դիր և ուսումնասիրիր 502-ը։
