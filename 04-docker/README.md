# 04 — Docker

## 1. What is Docker?

Docker-ը containerization platform է։

Container-ը application-ը և նրա runtime dependencies-ը մեկ isolated environment-ում աշխատեցնելու միջոց է։

---

## 2. Image vs Container

### Image

Image-ը immutable template/blueprint է, որից container են ստեղծվում։

### Container

Container-ը image-ի running կամ stopped instance է։

```text
Dockerfile
   ↓ build
Image
   ↓ run
Container
```

---

## 3. Docker vs VM

| | VM | Container |
|---|-----|-----------|
| Isolation | Full OS per VM | Shared kernel, isolated namespaces |
| Startup | Minutes | Seconds |
| Size | GB | MB (typical image) |
| Use case | Legacy, full OS need | Microservices, CI, portable apps |

Container-ը **kernel-ը share** է անում host-ի հետ; VM-ը առանձին guest OS ունի hypervisor-ի վրա։

---

## 4. Dockerfile

Dockerfile-ը image build-ի instructions է։

Օրինակ՝

```dockerfile
FROM node:22-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .
EXPOSE 3000
USER node
CMD ["node", "server.js"]
```

**Պատկերացում layer-ների մասին** — յուրաքանչյուր `RUN`, `COPY` նոր layer է; cache-ի համար dependencies-ը copy/install անել **առաջ**, source code-ը ** հետո**։

### Multi-stage build (interview)

```dockerfile
FROM node:22 AS build
WORKDIR /app
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
```

Վերջնական image-ը փոքր է — build tools-ը final image-ում չեն մնում։

---

## 5. Common commands

Images՝

```bash
docker images
```

Running containers՝

```bash
docker ps
```

All containers՝

```bash
docker ps -a
```

Logs՝

```bash
docker logs container_name
docker logs -f container_name
```

Container shell՝

```bash
docker exec -it container_name sh
```

Stop՝

```bash
docker stop container_name
```

Restart՝

```bash
docker restart container_name
```

Inspect՝

```bash
docker inspect container_name
```

Resource usage՝

```bash
docker stats
```

---

## 6. Port Mapping

```bash
docker run -p 8080:80 nginx
```

նշանակում է՝

```text
HOST :8080
     ↓
CONTAINER :80
```

Այսինքն՝

```text
http://localhost:8080
```

request-ը հասնում է container-ի port `80`-ին։

---

## 7. EXPOSE vs -p

`EXPOSE` Dockerfile-ում metadata/documentation է, որը նշում է intended port-ը։

```dockerfile
EXPOSE 80
```

`-p` իրական host-to-container port publishing է։

```bash
-p 8080:80
```

---

## 8. Volumes

Container-ի filesystem-ի տվյալները կարող են ephemeral լինել։

Volume-ը persistent data պահելու միջոց է։

Օրինակ՝ database-ի data-ն container restart/remove-ից հետո պահելու համար։

---

## 9. Docker Network

Containers-ը կարող են network-ով իրար հասնել։

Օրինակ՝

```text
nginx → app → postgres
```

Docker Compose-ում service name-ը կարող է օգտագործվել hostname-ի նման։

---

## 10. Docker Compose

Compose-ը multi-container application-ի configuration/orchestration-ի պարզ ձև է։

Օրինակ՝

```yaml
services:
  app:
    build: .
    ports:
      - "3000:3000"

  db:
    image: postgres
```

---

## 11. Healthcheck

Dockerfile կամ Compose-ում՝

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
  interval: 30s
  timeout: 5s
  retries: 3
```

Orchestrator-ը (Compose, Swarm, K8s) հասկանում է՝ container-ը **ապրո՞ծ** է, թե միայն process-ը running։

---

## 12. Cleanup

```bash
docker system df
docker image prune
docker container prune
docker system prune -a   # զգուշությամբ — unused images/containers
```

Disk 95% scenario-ում Docker images/layers-ը հաճախ մեծ տեղ են զբաղեցնում `/var/lib/docker`-ում։

---

## 13. Container-ը restart է լինում

Ստուգիր՝

```bash
docker ps -a
docker logs container_name
docker inspect container_name
```

Հնարավոր պատճառներ՝

- application crash
- wrong command
- missing environment variable
- dependency unavailable
- port/configuration issue

---

## 14. Application-ը container-ում աշխատում է, բայց host-ից չի բացվում

Ստուգիր՝

1. Container-ը running է։
2. Application-ը container-ի ներսում listening է։
3. Application-ը ճիշտ interface-ի վրա է listening։
4. Port mapping կա։
5. Firewall/network policy-ը թույլ է տալիս։
6. `docker inspect`-ով network/config-ը։

---

## Interview Questions

1. Image vs container?
2. Docker vs VM?
3. What is Dockerfile?
4. What is Compose?
5. What is a volume?
6. Explain `-p 8080:80`.
7. `EXPOSE` vs `-p`?
8. How do you view logs?
9. How do you enter a container?
10. Why might a container keep restarting?

## Practical Tasks

1. Run Nginx.
2. Publish it on port 8080.
3. View logs.
4. Enter the container.
5. Stop/restart it.
6. Create a simple Dockerfile.
7. Run an app + PostgreSQL using Compose.
