# 13 — Monitoring (Prometheus & Grafana)

## 1. Monitoring vs logging vs tracing

| Pillar | Question | Tools (examples) |
|--------|----------|------------------|
| **Metrics** | How much? How fast? Error rate? | Prometheus, CloudWatch |
| **Logs** | What exactly happened? | Loki, ELK, CloudWatch Logs |
| **Traces** | Where did latency go in microservices? | Jaeger, Tempo, X-Ray |

**Three pillars of observability** — metrics + logs + traces together։

Interview-ում՝ «alert on symptom (SLO), debug with logs/traces»։

---

## 2. Prometheus architecture

```text
Exporters / apps (metrics endpoint)
        ↓ scrape (pull)
Prometheus server (TSDB)
        ↓ PromQL
Grafana dashboards + Alertmanager → notifications
```

- **Pull model** — Prometheus HTTP scrape `/metrics`
- **TSDB** — time-series storage locally (or remote write)
- **PromQL** — query language
- **Alertmanager** — grouping, silencing, routing (PagerDuty, Slack)

---

## 3. Metrics types

| Type | Use |
|------|-----|
| **Counter** | Monotonically increasing (requests_total) |
| **Gauge** | Up/down (memory usage, queue depth) |
| **Histogram** | Distribution + buckets (latency) |
| **Summary** | Quantiles (client-side) |

Naming — `http_requests_total{method="GET",status="500"}`。

---

## 4. Exporters

Apps without native Prometheus format use **exporters**՝

- **node_exporter** — CPU, disk, memory (Linux host)
- **blackbox_exporter** — probe HTTP/TCP/DNS from outside
- **mysql_exporter**, **redis_exporter**, etc.

Kubernetes — **kube-state-metrics**, cAdvisor (container metrics), ServiceMonitor (Prometheus Operator)։

---

## 5. PromQL examples

```promql
# Request rate (5m window)
rate(http_requests_total[5m])

# Error ratio
sum(rate(http_requests_total{status=~"5.."}[5m]))
  /
sum(rate(http_requests_total[5m]))

# CPU usage %
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Disk almost full
node_filesystem_avail_bytes / node_filesystem_size_bytes < 0.1
```

**`rate()`** — per-second average over range for counters。

---

## 6. Grafana

- **Data sources** — Prometheus, Loki, CloudWatch, InfluxDB
- **Dashboards** — panels (graph, stat, table, heatmap)
- **Variables** — `$instance`, `$namespace` dropdowns
- **Alerts** (Grafana unified alerting) or Prometheus → Alertmanager

Dashboard design tips՝

- RED method — **Rate**, **Errors**, **Duration** (services)
- USE method — **Utilization**, **Saturation**, **Errors** (resources)

---

## 7. Alerting best practices

- Alert on **user-visible** or **SLO** symptoms, not every fluctuation
- Include **runbook** link in annotation
- **Severity** — page vs ticket
- Avoid alert fatigue — grouping in Alertmanager

Example rule (conceptual)՝

```yaml
groups:
  - name: web
    rules:
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m]))
            / sum(rate(http_requests_total[5m])) > 0.05
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: "Error rate above 5% for 10m"
```

---

## 8. SLI, SLO, SLA (brief)

- **SLI** — measured indicator (availability, latency p99)
- **SLO** — target (99.9% uptime)
- **SLA** — contract with customer

Error budget — how much downtime you can "spend" before feature freeze on reliability。

---

## 9. Kubernetes monitoring stack

Common pattern — **kube-prometheus-stack** (Helm)՝

- Prometheus Operator
- Grafana
- Alertmanager
- node-exporter, kube-state-metrics

Key views — pod CPU/memory, restarts, PVC usage, API server latency。

---

## 10. Troubleshooting with metrics

| Symptom | Metrics direction |
|---------|-------------------|
| Slow site | latency histogram, saturation (CPU, DB connections) |
| Errors spike | 5xx rate, upstream errors |
| Disk full | node_filesystem_*, volume metrics |
| Memory leak | container_memory_working_set_bytes trend |
| Pod flapping | kube_pod_container_status_restarts_total |

Correlate time range with **deployments** and **events**。

---

## 11. Prometheus limitations (interview)

- Single-server scale limits — federation, **Thanos**, **Mimir**, Cortex
- Pull — need network access to targets
- Cardinality explosion — too many unique label combinations
- Not long-term log storage — use Loki/ELK for logs

---

## Interview Questions

1. Pull vs push metrics model?
2. Counter vs gauge — examples?
3. What is `rate()` and why not use raw counter?
4. Prometheus vs Grafana — roles?
5. What is Alertmanager for?
6. RED vs USE?
7. How would you alert on disk filling up?
8. What causes high cardinality in Prometheus?
9. SLI vs SLO?
10. How do you monitor Kubernetes pods?
11. What is node_exporter?
12. Metrics vs logs — when which?

## Practical Tasks

1. Run Prometheus + Grafana locally (Docker Compose or kube-prometheus-stack doc)।
2. Add node_exporter target; build CPU/memory dashboard panel।
3. Write PromQL — 95th percentile latency from a histogram metric (conceptual bucket)।
4. Create one alert rule with `for: 5m` and explain why `for` matters।
5. Walk through debugging «latency up after deploy» using metrics + logs timeline।
