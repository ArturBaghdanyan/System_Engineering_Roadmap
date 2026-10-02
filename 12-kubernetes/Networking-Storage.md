## 4. Networking & Ingress (Երթևեկության Կառավարում)

### Ինչպես է աշխատում Ingress Controller-ը

Ingress-ը պարզապես **կանոնների (Rules)** YAML ֆայլ է։ Որպեսզի այն աշխատի, cluster-ում պետք է տեղադրված լինի **Ingress Controller** (օր.՝ NGINX Ingress Controller)։

```text
                        HTTP Request (example.com/api)
                                     │
                                     ▼
                        ┌────────────────────────┐
                        │   Load Balancer (Cloud)│
                        └────────────┬───────────┘
                                     │
                                     ▼
                        ┌────────────────────────┐
                        │  Ingress Controller    │
                        │  (NGINX Pod / Proxy)   │
                        └────────────┬───────────┘
                                     │
          ┌──────────────────────────┴──────────────────────────┐
          │ (Reads Ingress Rules & bypasses Service VIP directly│
          │  to target Pod IPs via EndpointSlices)              │
          ▼                                                     ▼
┌──────────────────┐                                  ┌──────────────────┐
│ Pod 1 (Backend)  │                                  │ Pod 2 (Backend)  │
│ IP: 10.244.1.15  │                                  │ IP: 10.244.2.40  │
└──────────────────┘                                  └──────────────────┘
```

**Կարևոր դետալ:** NGINX Ingress Controller-ը traffic-ը **չի ուղարկում Service-ի IP-ին**: Այն հետևում է K8s API-ի Endpoints/EndpointSlices-ին, ստանում է ռեալ **Pod-երի IP-ները** և traffic-ը Load Balance է անում **ուղիղ Pod-երին**։

---

## 5. Persistent Storage: PV, PVC & StorageClass

```text
 ┌────────────────────────────────────────────────────────┐
 │                      Developer                         │
 │  Creates PersistentVolumeClaim (PVC)                   │
 │  "I need 10GB of ReadWriteOnce storage"                │
 └───────────────────────────┬────────────────────────────┘
                             │
                             ▼
 ┌────────────────────────────────────────────────────────┐
 │                      StorageClass                      │
 │  Dynamic Provisioner (AWS EBS, GCP PD, Ceph)           │
 └───────────────────────────┬────────────────────────────┘
                             │ Creates automatically
                             ▼
 ┌────────────────────────────────────────────────────────┐
 │                      Cluster                           │
 │  Allocates PersistentVolume (PV) and binds to PVC      │
 └───────────────────────────┬────────────────────────────┘
                             │
                             ▼
 ┌────────────────────────────────────────────────────────┐
 │                        Pod                             │
 │  Mounts PVC as Volume inside Container                 │
 └────────────────────────────────────────────────────────┘
```

1. **StorageClass (SC):** Որոշում է, թե ինչ տեսակի storage պետք է ստեղծվի (օր. `gp3` AWS-ում)։ Ապահովում է **Dynamic Provisioning**։
2. **PersistentVolumeClaim (PVC):** Ծրագրավորողի հարցումն է Storage-ին («Ինձ պետք է 20Gi storage»):
3. **PersistentVolume (PV):** Ֆիզիկական Storage resource-ն է cluster-ում։

---
