## 2. Core Objects (Հիմնական Օբյեկտները)

### Pod

- K8s-ի **ամենափոքր** deployable միավորն է։
- Պարունակում է 1 կամ ավելի container-ներ, որոնք կիսում են **նույն Network Namespace-ը (նույն IP-ն)** և **Storage volume-ները**։
- _Multi-container Pod-ի օրինակ:_ Main app + Sidecar (լոգեր հավաքող կամ proxy անող container):

### Deployment & ReplicaSet

- Pod-երն անմիջապես (`kind: Pod`) չեն ստեղծվում production-ում, որովհետև դրանք սատկելուց հետո չեն վերականգնվում։
- **Deployment**-ը ղեկավարում է **ReplicaSet**-եր, իսկ ReplicaSet-ը պահում է Pod-երի ճիշտ քանակը։
- Ապահովում է Zero-downtime updates (Rolling Update) և Rollback (հետգլորում)։

### Service

Pod-երը transient են (մահանում են, նոր IP ստանում)։ Service-ը տալիս է **կայուն (Static) IP և DNS անուն** Pod-երի խմբի համար։

1. **ClusterIP** (Default). Հասանելի է **միայն cluster-ի ներսում**։
2. **NodePort**. Բացում է ֆիզիկական Node-ի static port (30000-32767 range-ում)։
3. **LoadBalancer**. Cloud provider-ից (AWS, GCP, Azure) պահանջում է External Load Balancer և տալիս է հանրային (Public) IP address։

### ConfigMap & Secret

- **ConfigMap:** Պահում է plain-text configuration-ներ (environment variables, config files):
- **Secret:** Պահում է sensitive տվյալներ (passwords, tokens, keys) base64 encode եղած տեսքով։

---

## 3. Pod-երի Կյանքի Ցիկլը (Lifecycle) և Probes

### Կարգավիճակները (Statuses)

- **Pending:** Pod-ը ընդունվել է K8s-ի կողմից, բայց container-ներից մեկը դեռ չի ստեղծվել։
- **Running:** Pod-ը կապվել է Node-ին, բոլոր container-ները ստեղծվել են, առնվազն 1-ը ընթացքի մեջ է։
- **Succeeded / Failed:** Ավարտվել է հաջողությամբ (CronJob) կամ սխալով։

### Սխալների Տեսակները (Troubleshooting Statuses)

- **CrashLoopBackOff:** Container-ը մեկնարկում է, սխալով փակվում, Kubelet-ը restart է անում, նորից է փակվում...
- **ImagePullBackOff / ErrImagePull:** Image-ի անունը/tag-ը սխալ է, կամ Private Registry authorization չկա։
- **OOMKilled (Out of Memory Killed):** Container-ը գերազանցել է իրեն հատկացված `limits.memory`-ն։

### Probes (Առողջության ստուգումներ)

```yaml
spec:
  containers:
    - name: my-app
      image: my-app:v1
      # 1. Startup Probe: Ստուգում է արդյոք ծրագիրը վերջնականապես RUN է եղել
      startupProbe:
        httpGet:
          path: /healthz
          port: 8080
        failureThreshold: 30
        periodSeconds: 10

      # 2. Liveness Probe: Ստուգում է՝ արդյոք ծրագիրը կենդանի է (եթե Deadlock է, Kubelet-ը RESTART կանի Pod-ը)
      livenessProbe:
        httpGet:
          path: /healthz
          port: 8080
        initialDelaySeconds: 15
        periodSeconds: 20

      # 3. Readiness Probe: Ստուգում է՝ արդյոք ծրագիրը պատրաստ է TRAFFIC ընդունելու
      readinessProbe:
        httpGet:
          path: /ready
          port: 8080
        initialDelaySeconds: 5
        periodSeconds: 10
```

---
