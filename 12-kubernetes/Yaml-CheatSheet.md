## 6. Ամբողջական, Պատրաստի YAML Օրինակ (`app.yaml`)

```yaml
# 1. PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-data-pvc
  namespace: default
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
---
# 2. ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "production"
  LOG_LEVEL: "info"
---
# 3. Secret (Base64 encoded)
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  DB_PASSWORD: c2VjcmV0a2V5MTIz
---
# 4. Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app.kubernetes.io/name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
        - name: web-container
          image: nginx:1.25-alpine
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "250m"
              memory: "256Mi"
          envFrom:
            - configMapRef:
                name: app-config
          env:
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: app-secret
                  key: DB_PASSWORD
          volumeMounts:
            - name: storage-volume
              mountPath: /usr/share/nginx/html/data
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 15
            periodSeconds: 20
      volumes:
        - name: storage-volume
          persistentVolumeClaim:
            claimName: app-data-pvc
---
# 5. Service
apiVersion: v1
kind: Service
metadata:
  name: web-app-service
spec:
  type: ClusterIP
  selector:
    app: web-app
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
---
# 6. Ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-app-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
    - host: myapp.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-app-service
                port:
                  number: 80
```

---

## 7. `kubectl` Հիմնական Հրամանները (Cheat Sheet)

```bash
# Diagnostic
kubectl get pods -o wide
kubectl describe pod <pod-name>
kubectl logs <pod-name> -c <container-name> --previous
kubectl exec -it <pod-name> -- /bin/sh

# Management
kubectl apply -f app.yaml
kubectl scale deployment web-app --replicas=5
kubectl rollout status deployment/web-app
kubectl rollout undo deployment/web-app
```

---

## 8. Troubleshooting Flowchart

```text
               1. kubectl get pods
                        │
      ┌─────────────────┼─────────────────┐
      ▼                 ▼                 ▼
   Pending      ImagePullBackOff   CrashLoopBackOff
      │                 │                 │
      ▼                 ▼                 ▼
describe pod      Check tag name    kubectl logs
(Node resource/   & registry        Check app error/
 Storage PVC issue) credentials      OOMKilled status
```

---

## 9. Գործնական Առաջադրանք (Step-by-Step Task)

1. **Մեկնարկեք Local Cluster:** `minikube start`
2. **Ստեղծեք Deployment & Service:** `kubectl apply -f app.yaml`
3. **Թեստավորեք Port-Forwarding-ը:** `kubectl port-forward svc/web-app-service 8080:80` -> բացեք `http://localhost:8080`:
4. **Սիմուլյացիա արեք Crash/Image Error:** Փոխեք image-ի tag-ը, տեսեք `ImagePullBackOff` status-ը, ապա արեք հետգլորում՝ `kubectl rollout undo deployment/web-app`։

---
