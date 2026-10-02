## 10. Հարցազրույցների Հարցեր և Պատասխաններ (Q&A)

### 1. Ի՞նչ է Pod-ը և ինչո՞վ է այն տարբերվում Container-ից։

- **Ի՞նչ է սա / Տարբերությունը:** Container-ը (օր. Docker container) անհատական process է isolation-ի մեջ։ Pod-ը Kubernetes-ի **ամենափոքր միավորն է**, որը պարունակում է **մեկ կամ մի քանի container-ներ**, որոնք աշխատում են նույն միջավայրում (shared network IP, storage)։
- **Ինչի՞ համար է օգտագործվում:** Հիմնական application-ը և դրան օգնող sidecar process-ները (օր.՝ log collector, proxy) միասին աշխատեցնելու համար։
- **Ի՞նչ խնդիր է լուծում:** Լուծում է իրար հետ խիստ կապված (tightly coupled) container-ների ցանցային և storage-ային կապի բարդությունը (Pod-ի ներսում container-ները իրար դիմում են `localhost`-ով)։

---

### 2. Deployment vs StatefulSet vs DaemonSet — Ո՞րն է տարբերությունը։

- **Ի՞նչ է սա / Տարբերությունը:**
  - **Deployment:** stateless (առանց state-ի) app-երի համար է (web apps, APIs)։ Pod-երը փոխարինելի են և չունեն ֆիքսված identity։
  - **StatefulSet:** stateful (տվյալ պահող) app-երի համար է (PostgreSQL, Kafka)։ Ամեն Pod ստանում է unique ֆիքսված անուն (`db-0`, `db-1`) և իրեն կապված persistent storage։
  - **DaemonSet:** Ապահովում է, որ **ամեն Node-ի վրա ճիշտ 1 Pod** աշխատի (օր. Datadog agent, Fluentd)։
- **Ինչի՞ համար է օգտագործվում:** Տարբեր տեսակի architecture ունեցող ծրագրերը ճիշտ կառավարելու համար։
- **Ի՞նչ խնդիր է լուծում:** Ստանդարտ Deployment-ով հնարավոր չէր ճիշտ scale անել տվյալների բազաները կամ cluster-ի ամբողջական monitoring/logging իրականացնել։

---

### 3. ClusterIP vs NodePort vs LoadBalancer — Ո՞րն է տարբերությունը։

- **Ի՞նչ է սա / Տարբերությունը:**
  - **ClusterIP:** Ներքին Virtual IP (հասանելի է միայն cluster-ի ներսից)։
  - **NodePort:** Բացում է ֆիզիկական Node-ի static port (30000-32767) External access-ի համար։
  - **LoadBalancer:** Ստեղծում է cloud-ի external load balancer (AWS ALB, GCP LB) և տալիս է public IP։
- **Ինչի՞ համար է օգտագործվում:** Traffic-ը դեպի Pod-եր ուղղորդելու համար։
- **Ի՞նչ խնդիր է լուծում:** Pod-երը կարող են մահանալ և փոխել IP-ները, իսկ Service-ը ապահովում է static endpoint (IP + DNS):

---

### 4. Ի՞նչ է Liveness Probe-ը և Readiness Probe-ը։

- **Ի՞նչ է սա / Տարբերությունը:**
  - **Liveness Probe:** Ստուգում է, թե արդյոք container-ը **կենդանի է**։ Եթե probe-ը failure տա, Kubelet-ը **սպանում և restart է անում** Pod-ը։
  - **Readiness Probe:** Ստուգում է, թե արդյոք container-ը **պատրաստ է traffic ընդունելու**։ Եթե failure տա, Pod-ը **ժամանակավորապես հանվում է** Service endpoints-ից (չի restart-վում)։
- **Ինչի՞ համար է օգտագործվում:** Application health status-ը ավտոմատ կառավարելու համար։
- **Ի՞նչ խնդիր է լուծում:** Կանխում է Deadlock-երը (Liveness) և կանխում է traffic-ի ուղղորդումը դեպի դեռ չբեռնավորված/slow-start app-եր (Readiness)։

---

### 5. Ի՞նչ է կատարվում, երբ Node-ը մահանում է (Node Failure)։

- **Ի՞նչ է սա / Պատասխանը:** Control Plane-ը (`kube-controller-manager`) նկատում է, որ Node-ը դադարել է Heartbeat ուղարկել (`node-monitor-grace-period`, default 40 վայրկյան)։
- **Ինչի՞ համար է օգտագործվում:** Cluster self-healing (ինքնավերականգնման) մեխանիզմ։
- **Ի՞նչ խնդիր է լուծում:** Մահացած Node-ի Pod-երը ավտոմատ rescheduling են լինում (տեղափոխվում են) ուրիշ առողջ Node-երի վրա, ապահովելով High Availability (HA)։

---

### 6. Ինչպե՞ս է աշխատում Rolling Update-ը։

- **Ի՞նչ է սա / Պատասխանը:** Deployment strategy է, որը թարմացնում է Pod-երը **աստիճանաբար** (սկզբում ստեղծվում է 1 նոր Pod, երբ ready է դառնում, սպանվում է 1 հին Pod)։
- **Ինչի՞ համար է օգտագործվում:** Ծրագրի նոր տարբերակը (Image tag) deploy անելու համար։
- **Ի՞նչ խնդիր է լուծում:** Վերացնում է downtime-ը (Zero Downtime Deployment)։ Օգտատերերը չեն զգում, որ համակարգը թարմացվել է։

---

### 7. ConfigMap vs Secret — Ո՞րն է տարբերությունը։

- **Ի՞նչ է սա / Տարբերությունը:**
  - **ConfigMap:** Plain-text config-ների համար է (ENV vars, appsettings.json)։
  - **Secret:** Sensitive տվյալների համար է (DB password, API token)։ Պահվում է Base64 encode եղած։
- **Ինչի՞ համար է օգտագործվում:** Configuration-ը application code-ից (Docker image-ից) անջատելու համար (12-Factor App methodology):
- **Ի՞նչ խնդիր է լուծում:** Կանխում է passwords/keys-երը Docker image-ների կամ Git repository-ի մեջ hardcode անելը։

---

### 8. Ի՞նչ է Ingress-ը և Ingress Controller-ը։

- **Ի՞նչ է սա / Տարբերությունը:** Ingress-ը YAML resource է (կանոնների ցուցակ), իսկ Ingress Controller-ը (օր. Nginx) այն Pod-ն է/Proxy-ն է, որը իրականում կատարում է այդ routing-ը։
- **Ինչի՞ համար է օգտագործվում:** 1 հատ Load Balancer-ի միջոցով տասնյակ microservice-ներ HTTP/HTTPS Host-երով (`api.domain.com`, `app.domain.com`) կամ Path-երով (`/api`, `/web`) դուրս բերելու համար։
- **Ի՞նչ խնդիր է լուծում:** Խնայում է գումար (ամեն Service-ի համար առանձին expensive Cloud Load Balancer չբացելու համար) և ղեկավարում է TLS/SSL termination-ը։

---

### 9. Resources Requests vs Limits — Ո՞րն է տարբերությունը։

- **Ի՞նչ է սա / Տարբերությունը:**
  - **Requests:** Մինիմալ CPU/RAM-ն է, որը **երաշխավորված է** Pod-ի համար։ Scheduler-ը սրանով է որոշում, թե որ Node-ում տեղավորի Pod-ը։
  - **Limits:** Մաքսիմալ CPU/RAM-ն է, որից ավել Pod-ը չի կարող սպառել։
- **Ինչի՞ համար է օգտագործվում:** Cluster-ի ռեսուրսները ճիշտ բաշխելու համար։
- **Ի՞նչ խնդիր է լուծում:** Կանխում է "Noisy Neighbor" խնդիրը (երբ 1 Pod-ը ուտում է Node-ի ամբողջ RAM/CPU-ն և սպանում կողքի Pod-երին)։

---

### 10. Ինչպե՞ս եք դեբագ (Debug) անում CrashLoopBackOff-ը։

- **Ի՞նչ է սա / Քայլերը:**
  1. `kubectl get pods` — տեսնել քանի restart է եղել։
  2. `kubectl logs <pod-name> --previous` — **ամենակարևոր քայլը**՝ տեսնելու սպանված container-ի վերջին սխալի լոգը։
  3. `kubectl describe pod <pod-name>` — ստուգել Events, Exit Code-ը (օր. Exit Code 137 = OOMKilled), Liveness probe failures-ը։
  4. Եթե լոգերում ոչինչ չկա, `kubectl exec` կամ փոխել `command: ["sleep", "3600"]` Pod-ի ներս մտնելու համար։
- **Ի՞նչ խնդիր է լուծում:** Արագ գտնել application crash-ի բուն պատճառը (Root Cause):

---

### 11. Ինչի՞ համար է օգտագործվում Namespace-ը։

- **Ի՞նչ է սա:** Virtual isolation (տարանջատում) Cluster-ի ներսում (`dev`, `staging`, `prod`):
- **Ինչի՞ համար է օգտագործվում:** Թիմերի, միջավայրերի կամ microservice-ների խմբերը իրարից առանձնացնելու համար։
- **Ի՞նչ խնդիր է լուծում:** Կանխում է անունների բախումը (Name collisions), թույլ է տալիս դնել ResourceQuotas (օր. Dev environment-ին տալ max 10CPU) և RBAC access rights (Security):

---

### 12. Ի՞նչ է Helm-ը և ինչո՞ւ օգտագործել այն։

- **Ի՞նչ է սա:** Package Manager Kubernetes-ի համար (ինչպես `apt`, `npm`, `pip`):
- **Ինչի՞ համար է օգտագործվում:** YAML ֆայլերը template-ավորելու (`charts`) և տարբեր environment-ների համար `values.yaml`-ով parametrable դարձնելու համար։
- **Ի՞նչ խնդիր է լուծում:** Վերացնում է հարյուրավոր static YAML ֆայլեր ձեռքով կառավարելու բարդությունը, ապահովում է 1-command deployment (`helm install my-app`) և 1-command rollback (`helm rollback`):
