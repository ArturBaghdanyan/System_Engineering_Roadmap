# 12 — Kubernetes (K8s)

## 1. Ինչու Kubernetes

Container orchestration — deploy, scale, heal, network, secrets across many nodes։

```text
Control plane (API, scheduler, etcd)
        ↓
Worker nodes (kubelet, container runtime)
        ↓
Pods (one or more containers)
```

Managed services — **EKS**, **AKS**, **GKE**։

---

## 2. Core objects

| Object | Role |
|--------|------|
| **Pod** | Smallest deploy unit; shared network namespace |
| **Deployment** | Declarative replicas, rolling updates |
| **ReplicaSet** | Keeps N pods (usually via Deployment) |
| **Service** | Stable IP/DNS to pods (ClusterIP, NodePort, LoadBalancer) |
| **Ingress** | HTTP routing, TLS (needs Ingress controller) |
| **ConfigMap** | Non-secret config |
| **Secret** | Sensitive data (base64 at rest — still protect etcd/RBAC) |
| **Namespace** | Logical isolation |
| **StatefulSet** | Stable identity, ordered deploy (DB) |
| **DaemonSet** | One pod per node (agents, log collectors) |
| **Job / CronJob** | Batch / scheduled |

---

## 3. Pod lifecycle (brief)

```text
Pending → Running → Succeeded / Failed
```

- **CrashLoopBackOff** — app exits, kubelet restarts
- **ImagePullBackOff** — wrong tag, registry auth
- **Pending** — no resources, scheduling, PVC not bound

---

## 4. Deployment և Service (minimal YAML)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx:1.25
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
```

**Labels/selectors** — Service finds Pods։

---

## 5. Probes

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
```

- **Liveness** — restart if deadlocked
- **Readiness** — remove from Service endpoints until ready
- **Startup** — slow-start apps

---

## 6. Resources

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```

**Requests** — scheduling; **limits** — cap (OOM kill if memory exceeded)։

---

## 7. Networking

- Pod IP — cluster internal
- **Service** — virtual IP, kube-proxy / iptables or eBPF
- **DNS** — `web.default.svc.cluster.local`
- **NetworkPolicy** — firewall between pods (needs CNI support)
- **Ingress** — host/path routing to services

```text
Internet → LB → Ingress → Service → Pods
```

---

## 8. Storage

- **emptyDir** — ephemeral
- **PersistentVolume (PV)** + **PersistentVolumeClaim (PVC)** — durable
- **StorageClass** — dynamic provisioning (EBS, etc.)

StatefulSet + PVC — databases (with ops caution)։

---

## 9. kubectl essentials

```bash
kubectl get pods -A
kubectl describe pod web-xxx
kubectl logs web-xxx -c nginx --previous
kubectl exec -it web-xxx -- /bin/sh
kubectl apply -f deployment.yaml
kubectl rollout status deployment/web
kubectl rollout undo deployment/web
kubectl get events --sort-by=.metadata.creationTimestamp
```

**Context/namespace** — `-n prod`, `kubectl config use-context`։

---

## 10. Troubleshooting flow

1. `kubectl get pods` — status, restarts
2. `describe pod` — events, probe failures, PVC
3. `logs` — application error
4. `exec` — curl inside cluster, DNS check
5. Service endpoints — `kubectl get ep`
6. Ingress — rules, backend, cert

Common interview scenario — **CrashLoopBackOff**, **502 via Ingress**, **ImagePullBackOff**。

---

## 11. Helm (overview)

Package manager for K8s — chart = templates + values。

```bash
helm install myapp ./chart -f values-prod.yaml
helm upgrade --install ...
```

---

## 12. Security basics

- **RBAC** — Role, RoleBinding, ServiceAccount
- **Pod Security** / securityContext — non-root, read-only root FS
- **Secrets** — external secret operators, vault integration
- **Private registry** — `imagePullSecrets`

---

## Interview Questions

1. Pod vs container?
2. Deployment vs StatefulSet?
3. ClusterIP vs NodePort vs LoadBalancer?
4. Liveness vs readiness probe?
5. What happens when a node dies?
6. How does rolling update work?
7. ConfigMap vs Secret?
8. What is Ingress?
9. requests vs limits?
10. How do you debug CrashLoopBackOff?
11. What is a namespace used for?
12. Helm — why use it?

## Practical Tasks

1. Local cluster (minikube/kind/Docker Desktop) — deploy nginx Deployment + Service।
2. Port-forward — `kubectl port-forward svc/web 8080:80` and test।
3. Break liveness probe — observe restarts in `kubectl get pods`।
4. Scale — `kubectl scale deployment/web --replicas=5`।
5. Rollout undo after bad image tag change।
