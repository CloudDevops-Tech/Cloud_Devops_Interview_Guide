# 📊 TOPIC: OBSERVABILITY

> Monitoring vs observability, metrics/logs/traces, Prometheus & PromQL, Grafana & Alertmanager, SLI/SLO/SLA, golden signals, alerting, logging stacks, tracing and incident scenarios.

<br>

---

### Q1. What is the difference between Monitoring and Observability?

#### 📌 Short Explanation
A foundational question. Interviewers want to see that you understand monitoring answers *known* questions, while observability helps you investigate *unknown* problems.

#### ✅ Answer
- **Monitoring** tells you **that** something is wrong (CPU is 95%, error rate is up) using predefined metrics and alerts.
- **Observability** helps you understand **why** it is wrong by exploring metrics, logs, and traces without having to predict the failure in advance.

#### 📖 Detailed Explanation

| Aspect | Monitoring | Observability |
|--------|-----------|---------------|
| **Question answered** | "Is the system healthy?" | "Why is the system behaving this way?" |
| **Approach** | Predefined dashboards and thresholds | Ad-hoc exploration of rich telemetry |
| **Handles** | Known failure modes ("known-knowns") | Unknown failure modes ("unknown-unknowns") |
| **Data** | Mostly metrics | Metrics + Logs + Traces (+ events, profiles) |
| **Example** | Alert: "5xx rate > 5%" | Trace shows one slow DB query in the checkout service causing the 5xx |

> **Key takeaway:** Monitoring is a **subset** of observability. You monitor to detect; you use observability to diagnose.

<br>

---

### Q2. What are the three pillars of observability? Explain with examples.

#### 📌 Short Explanation
Tests whether you know which signal to use for which problem.

#### ✅ Answer
The three pillars are **Metrics**, **Logs**, and **Traces**.

#### 📖 Detailed Explanation

| Pillar | What it is | Best for | Tools |
|--------|-----------|----------|-------|
| **Metrics** | Numeric time-series values (CPU, request rate, latency) | Trends, alerting, capacity planning | Prometheus, CloudWatch, Datadog |
| **Logs** | Timestamped records of discrete events | Detailed root-cause debugging | ELK, Loki, CloudWatch Logs, Splunk |
| **Traces** | End-to-end path of a single request across services | Finding which service or call is slow | Jaeger, Tempo, AWS X-Ray, Zipkin |

**🔹 How they work together**
1. A **metric** alert fires: p95 latency is high.
2. A **trace** shows the slowness is in the `payment-service → DB` call.
3. **Logs** from `payment-service` for that trace ID reveal a connection-pool timeout.

> 💡 Link them with a common **trace ID / correlation ID** present in logs and traces.

<br>

---

### Q3. Explain the Prometheus architecture. Why does Prometheus use a pull model?

#### 📌 Short Explanation
Prometheus is the de-facto metrics standard in Kubernetes environments. Interviewers expect you to know its components and data flow.

#### ✅ Answer
Prometheus **scrapes (pulls)** metrics over HTTP from targets at regular intervals, stores them in a local time-series database (TSDB), evaluates rules, and sends alerts to **Alertmanager**. Grafana is typically used for visualization.

#### 📖 Detailed Explanation

**🔹 Core components**

| Component | Role |
|-----------|------|
| **Prometheus Server** | Scrapes targets, stores TSDB data, evaluates rules, serves PromQL |
| **Exporters** | Expose metrics for systems that don't natively do it (node_exporter, mysqld_exporter) |
| **Service Discovery** | Finds targets dynamically (Kubernetes, EC2, Consul) |
| **Pushgateway** | Accepts metrics from short-lived batch jobs |
| **Alertmanager** | Deduplicates, groups, routes, and silences alerts |
| **Grafana** | Dashboards and visualization |

**🔹 Flow**
```text
Targets (/metrics) <--scrape-- Prometheus --alerts--> Alertmanager --> Slack / PagerDuty / Email
                                    |
                                    +--PromQL--> Grafana
```

**🔹 Why pull?**
- Prometheus **knows if a target is down** (a failed scrape sets `up == 0`)
- Easier to run multiple Prometheus servers against the same targets
- Targets stay simple: just expose `/metrics`
- You can manually `curl` a target's metrics endpoint to debug

**🔹 When push is needed**
Short-lived jobs (cron, batch) may finish before being scraped, so they push to the **Pushgateway**.

<br>

---

### Q4. What are the Prometheus metric types? Explain Counter, Gauge, Histogram, and Summary.

#### 📌 Short Explanation
Choosing the wrong metric type leads to wrong graphs and wrong alerts. This is a common differences question.

#### ✅ Answer
- **Counter** — only goes up (resets on restart)
- **Gauge** — goes up and down
- **Histogram** — samples observations into configurable buckets
- **Summary** — calculates quantiles on the client side

#### 📖 Detailed Explanation

| Type | Behavior | Example | Typical query |
|------|----------|---------|---------------|
| **Counter** | Monotonically increasing | `http_requests_total` | `rate(http_requests_total[5m])` |
| **Gauge** | Can increase or decrease | `memory_usage_bytes`, `queue_length` | Use value directly |
| **Histogram** | Buckets + `_sum` + `_count` | `http_request_duration_seconds` | `histogram_quantile(0.95, ...)` |
| **Summary** | Pre-computed quantiles | `rpc_duration_seconds{quantile="0.99"}` | Read quantile directly |

**🔹 Histogram vs Summary**

| Aspect | Histogram | Summary |
|--------|-----------|---------|
| Quantile calculated | Server side (PromQL) | Client side |
| Aggregation across instances | ✅ Yes | ❌ No (quantiles can't be averaged) |
| Flexibility | Can change quantile later | Fixed at instrumentation time |

> **Key takeaway:** Prefer **histograms** in distributed systems because they can be aggregated across pods.

<br>

---

### Q5. Write PromQL queries for request rate, error percentage, and p95 latency.

#### 📌 Short Explanation
A practical question to check hands-on PromQL ability.

#### ✅ Answer
Use `rate()` on counters, divide for ratios, and `histogram_quantile()` for percentiles.

#### 📖 Detailed Explanation

**🔹 Requests per second (per service)**
```promql
sum(rate(http_requests_total[5m])) by (service)
```

**🔹 Error percentage (5xx)**
```promql
100 *
sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)
/
sum(rate(http_requests_total[5m])) by (service)
```

**🔹 p95 latency**
```promql
histogram_quantile(0.95,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service))
```

**🔹 Pods using more than 80% of memory limit**
```promql
container_memory_working_set_bytes / on(pod) kube_pod_container_resource_limits{resource="memory"} > 0.8
```

**🔹 `rate` vs `irate` vs `increase`**

| Function | Use |
|----------|-----|
| `rate()` | Per-second average over the range — best for alerts and dashboards |
| `irate()` | Instant rate from the last two samples — good for fast-moving graphs |
| `increase()` | Total increase over the range |

<br>

---

### Q6. What is the difference between Prometheus and Grafana? Where does Alertmanager fit?

#### 📌 Short Explanation
Many candidates confuse the roles. This checks whether you understand each tool's responsibility.

#### ✅ Answer
- **Prometheus** collects, stores, and queries metrics, and evaluates alert rules.
- **Grafana** visualizes data from many sources (Prometheus, Loki, CloudWatch, Elasticsearch).
- **Alertmanager** receives alerts from Prometheus and handles notification routing.

#### 📖 Detailed Explanation

| Tool | Responsibility | Stores data? |
|------|---------------|--------------|
| **Prometheus** | Scrape, store, query, evaluate alert rules | ✅ Yes (TSDB) |
| **Grafana** | Dashboards, visual exploration, unified alerting | ❌ No (reads from data sources) |
| **Alertmanager** | Group, deduplicate, silence, route alerts | ❌ Only alert state |

**🔹 Alertmanager features**
- **Grouping** — many alerts become one notification
- **Inhibition** — suppress "pod down" alerts if "node down" is already firing
- **Silencing** — mute alerts during maintenance
- **Routing** — critical → PagerDuty, warning → Slack

**🔹 Sample alert rule**
```yaml
groups:
- name: app-alerts
  rules:
  - alert: HighErrorRate
    expr: sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) > 0.05
    for: 10m
    labels:
      severity: critical
    annotations:
      summary: "5xx error rate above 5% for 10 minutes"
```

<br>

---

### Q7. What are SLI, SLO, and SLA? What is an error budget?

#### 📌 Short Explanation
Tests Site Reliability Engineering (SRE) fundamentals and your ability to link monitoring to business goals.

#### ✅ Answer
- **SLI** — the measurement (e.g., % of successful requests)
- **SLO** — the internal target for that measurement (e.g., 99.9%)
- **SLA** — the contractual promise to customers, with penalties
- **Error budget** — the allowed unreliability = `100% − SLO`

#### 📖 Detailed Explanation

| Term | Meaning | Example |
|------|---------|---------|
| **SLI** | Service Level **Indicator** | Successful requests / total requests |
| **SLO** | Service Level **Objective** | 99.9% success over 30 days |
| **SLA** | Service Level **Agreement** | Refund credits if availability < 99.5% |

**🔹 Error budget example**
- SLO = 99.9% over 30 days → budget = 0.1% ≈ **43 minutes** of downtime per month
- Budget remaining → ship features faster
- Budget exhausted → freeze risky releases, focus on reliability

**🔹 Rule of thumb:** SLO should be **stricter** than SLA so you can react before breaching the contract.

<br>

---

### Q8. What are the Four Golden Signals? Explain the difference between the RED and USE methods.

#### 📌 Short Explanation
Interviewers use this to see if you have a structured approach to deciding *what* to monitor.

#### ✅ Answer
- **Golden Signals (Google SRE):** Latency, Traffic, Errors, Saturation
- **RED method:** Rate, Errors, Duration — for **services/requests**
- **USE method:** Utilization, Saturation, Errors — for **resources** (CPU, disk, network)

#### 📖 Detailed Explanation

| Method | Focus | Signals | Example |
|--------|-------|---------|---------|
| **Golden Signals** | Any user-facing system | Latency, Traffic, Errors, Saturation | API gateway dashboard |
| **RED** | Microservices | Rate, Errors, Duration | Requests/sec, 5xx %, p95 latency of `orders` service |
| **USE** | Infrastructure | Utilization, Saturation, Errors | CPU %, run-queue length, disk errors on a node |

> **Key takeaway:** Use **RED for services** and **USE for infrastructure**. Together they cover both the application and the platform.

<br>

---

### Q9. Scenario: You get a 2 AM alert that API latency is high. Walk through your investigation.

#### 📌 Short Explanation
A classic incident-response scenario. Interviewers want a calm, structured, data-driven approach.

#### ✅ Answer
Confirm impact → check recent changes → narrow down using metrics → traces → logs → mitigate first → root cause later.

#### 📖 Detailed Explanation

**🔹 Step 1: Assess impact**
- Which endpoints and what percentage of users? Check the RED dashboard.

**🔹 Step 2: Check recent changes**
- New deployment, config change, feature flag, traffic spike, or cloud event?

**🔹 Step 3: Narrow down using metrics**
```promql
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service))
```
- Is it one service or all? Compare with CPU, memory, DB connections, and queue depth.

**🔹 Step 4: Use traces**
- Open slow traces in Jaeger/Tempo/X-Ray and find the slowest span (DB, external API, cache).

**🔹 Step 5: Check logs**
```bash
kubectl logs deploy/payment-service --since=15m | grep -i -E "timeout|error"
```

**🔹 Step 6: Mitigate**
- Roll back the deployment, scale out, restart a stuck pod, or disable the faulty feature flag.

**🔹 Step 7: Post-incident**
- Write a blameless postmortem, add missing alerts/dashboards, and create action items.

> **Key takeaway:** **Mitigate first, investigate later.** Restore service, then find the root cause.

<br>

---

### Q10. Scenario: Your team is suffering from alert fatigue. How do you fix it?

#### 📌 Short Explanation
Tests your maturity in designing alerts that are actionable rather than noisy.

#### ✅ Answer
Alert on **symptoms** (user impact), not every cause; make each alert **actionable**; add `for:` durations, grouping, and severity routing; review alerts regularly.

#### 📖 Detailed Explanation

| Problem | Fix |
|---------|-----|
| Alerts fire on short spikes | Add `for: 5m–10m` |
| Too many duplicate alerts | Use Alertmanager **grouping** and **inhibition** |
| Alerts with no action needed | Delete them or downgrade to dashboard-only |
| Everything pages the on-call | Route by **severity** (page vs ticket vs Slack) |
| Static thresholds fail | Use SLO **burn-rate** alerts |
| No ownership | Add `team` labels and runbook links in annotations |

**🔹 Good alert checklist**
- ✅ Does it indicate user impact?
- ✅ Is there a clear action?
- ✅ Does it link to a runbook?
- ✅ Is it reviewed monthly?

**🔹 Cause vs symptom**
- ❌ Page for "CPU > 80%" (may not hurt users)
- ✅ Page for "error rate > 5% for 10 minutes"

<br>

---

### Q11. What is the difference between the ELK stack and Loki (PLG stack)?

#### 📌 Short Explanation
Log aggregation choices affect cost and performance. This is a common tool-comparison question.

#### ✅ Answer
- **ELK** (Elasticsearch, Logstash, Kibana) **fully indexes log content**, giving powerful search at higher cost.
- **Loki** indexes only **labels** and stores log lines compressed, making it cheaper and simpler, and it integrates natively with Grafana.

#### 📖 Detailed Explanation

| Aspect | ELK / EFK | Loki (Promtail + Loki + Grafana) |
|--------|-----------|----------------------------------|
| **Indexing** | Full-text index of every field | Labels only (like Prometheus) |
| **Storage cost** | High | Low (object storage such as S3) |
| **Query language** | KQL / Lucene | LogQL |
| **Search power** | Excellent full-text analytics | Good for filtering by labels, then grep |
| **Operations** | Heavier (JVM, shard management) | Lightweight |
| **Best for** | Complex log analytics, security/SIEM | Kubernetes logs correlated with metrics |

**🔹 LogQL example**
```logql
{app="payment", namespace="prod"} |= "error" | json | status >= 500
```

**🔹 Log shippers**
- **Fluent Bit / Fluentd / Filebeat** — ELK/EFK
- **Promtail / Grafana Alloy** — Loki

> 💡 Always use **structured (JSON) logging** with trace IDs and avoid logging secrets or PII.

<br>

---

### Q12. What is distributed tracing? Explain OpenTelemetry, trace, and span.

#### 📌 Short Explanation
In microservices, one user request can touch many services. Tracing shows the whole journey.

#### ✅ Answer
**Distributed tracing** follows a single request across services. A **trace** is the full journey; a **span** is one unit of work within it. **OpenTelemetry (OTel)** is the vendor-neutral standard for generating and exporting metrics, logs, and traces.

#### 📖 Detailed Explanation

**🔹 Concepts**

| Term | Meaning |
|------|---------|
| **Trace** | End-to-end record of one request (has a Trace ID) |
| **Span** | A single operation (HTTP call, DB query) with start time, duration, and tags |
| **Context propagation** | Passing the trace ID between services via headers (e.g., W3C `traceparent`) |
| **Sampling** | Recording only a percentage of traces to control cost |

**🔹 Example trace**
```text
GET /checkout (320 ms)
 ├─ auth-service         (20 ms)
 ├─ cart-service         (40 ms)
 └─ payment-service      (240 ms)
     └─ SELECT orders... (210 ms)  ← bottleneck
```

**🔹 OpenTelemetry pipeline**
```text
App (OTel SDK) → OTel Collector → Jaeger / Tempo / X-Ray / Datadog
```
- The **Collector** receives, processes (batch, filter), and exports telemetry to any backend, avoiding vendor lock-in.

**🔹 Sampling types**
- **Head-based:** decide at the start of the request
- **Tail-based:** decide after the trace is complete (keep slow/error traces)

<br>

---

### Q13. Scenario: Prometheus memory usage keeps growing and it eventually crashes. What could be the cause?

#### 📌 Short Explanation
The classic cause is **high cardinality**. This tests your understanding of how Prometheus stores data.

#### ✅ Answer
Most likely **high-cardinality labels** (user IDs, request IDs, full URLs) creating millions of unique time series. Fix by removing those labels, relabeling, and limiting retention and scrape scope.

#### 📖 Detailed Explanation

**🔹 What is cardinality?**
Each unique combination of label values is a separate time series. `http_requests_total{user_id="..."}` with 1M users creates 1M series.

**🔹 Investigate**
```promql
topk(10, count by (__name__)({__name__=~".+"}))
```
```bash
# Prometheus UI → Status → TSDB Status shows top series and label cardinality
```

**🔹 Fixes**
- Remove unbounded labels (`user_id`, `session_id`, `ip`, raw URL paths)
- Drop metrics/labels using `metric_relabel_configs`
- Reduce scrape targets or increase scrape interval
- Set sensible `--storage.tsdb.retention.time`
- Set `sample_limit` per scrape job
- Use recording rules for expensive queries
- Shard Prometheus or move to a scalable backend (Thanos, Mimir, VictoriaMetrics)

```yaml
metric_relabel_configs:
  - action: labeldrop
    regex: user_id
```

> **Key takeaway:** Labels should have **bounded, low-cardinality** values.

<br>

---

### Q14. Scenario: A target shows as DOWN in Prometheus (or isn't scraped at all). How do you troubleshoot?

#### 📌 Short Explanation
Tests your knowledge of the scrape pipeline: discovery → relabeling → network → endpoint.

#### ✅ Answer
Check **Status → Targets** in the Prometheus UI, then verify service discovery/labels, network reachability, the `/metrics` endpoint, and authentication.

#### 📖 Detailed Explanation

**🔹 Step 1: Look at the error in Status → Targets**
- `connection refused` → app not listening / wrong port
- `context deadline exceeded` → timeout, firewall, or slow endpoint
- `401/403` → auth or TLS config missing
- `server returned HTTP status 404` → wrong `metrics_path`

**🔹 Step 2: Target not listed at all**
- Check service discovery config, `ServiceMonitor`/`PodMonitor` label selectors (Prometheus Operator), and `relabel_configs` (a `keep`/`drop` rule may remove it)

**🔹 Step 3: Test connectivity from inside the cluster**
```bash
kubectl exec -it prometheus-0 -- wget -qO- http://my-service:8080/metrics | head
```

**🔹 Step 4: Check NetworkPolicy, Security Groups, and port names** (service port name must match `ServiceMonitor` endpoint port)

**🔹 Step 5: Use the `up` metric**
```promql
up{job="my-app"} == 0
```

<br>

---

### Q15. How do you make Prometheus highly available and store data long term?

#### 📌 Short Explanation
A single Prometheus server has local storage and is a single point of failure. This checks your knowledge of scaling observability.

#### ✅ Answer
Run **two identical Prometheus replicas** scraping the same targets for HA, and use **Thanos, Cortex/Mimir, or VictoriaMetrics** for long-term storage and a global query view.

#### 📖 Detailed Explanation

| Option | How it works |
|--------|-------------|
| **HA pair** | Two Prometheus servers scrape the same targets; Alertmanager deduplicates alerts |
| **Thanos** | Sidecar uploads TSDB blocks to S3/GCS; Querier gives a global view; Compactor downsamples |
| **Mimir / Cortex** | Prometheus `remote_write` into a horizontally scalable, multi-tenant backend |
| **Federation** | A global Prometheus scrapes selected metrics from regional Prometheus servers |

**🔹 Remote write example**
```yaml
remote_write:
  - url: https://mimir.example.com/api/v1/push
```

**🔹 Key points**
- Local TSDB is not meant for years of retention
- Use **downsampling** for long-term queries
- Run Alertmanager as a **cluster** (gossip) for HA

<br>

---

### Q16. Scenario: How would you monitor a Kubernetes cluster end to end?

#### 📌 Short Explanation
Tests whether you know which components expose which metrics and how to cover cluster, node, pod, and application layers.

#### ✅ Answer
Use **kube-prometheus-stack** (Prometheus Operator, Grafana, Alertmanager) with **node-exporter**, **kube-state-metrics**, and **cAdvisor**, plus Loki for logs and OpenTelemetry/Jaeger for traces.

#### 📖 Detailed Explanation

| Layer | What to monitor | Source |
|-------|----------------|--------|
| **Node** | CPU, memory, disk, network | `node-exporter` |
| **Container / Pod** | CPU, memory usage, throttling, restarts | `cAdvisor` (kubelet) |
| **Kubernetes objects** | Pod phase, Deployment replicas, HPA, PVC status | `kube-state-metrics` |
| **Control plane** | API server latency, etcd health, scheduler | Control plane metrics endpoints |
| **Application** | RED metrics, business KPIs | App `/metrics` |
| **Logs** | Container stdout/stderr | Fluent Bit / Promtail → Loki/ELK |

**🔹 Install**
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring --create-namespace
```

**🔹 Key alerts to create**
- Pod `CrashLoopBackOff` or high restart count
- Node `NotReady`
- PVC almost full
- Deployment replicas unavailable
- CPU throttling and OOMKilled containers
- API server error rate

> 💡 **kube-state-metrics** exposes the *state* of objects; **node-exporter** and **cAdvisor** expose *resource usage*.

<br>

---

### Q17. What is the difference between synthetic monitoring, RUM, and APM?

#### 📌 Short Explanation
Tests awareness of monitoring from the user's perspective, not just from the server side.

#### ✅ Answer
- **Synthetic monitoring** uses scripted probes to test availability from outside.
- **RUM (Real User Monitoring)** collects performance data from real users' browsers.
- **APM (Application Performance Monitoring)** instruments application code for transactions, traces, and errors.

#### 📖 Detailed Explanation

| Type | Data source | Strength | Example tools |
|------|-------------|----------|---------------|
| **Synthetic** | Scheduled scripted checks | Detects outages before users do; 24x7 coverage | Prometheus Blackbox Exporter, CloudWatch Synthetics, Pingdom |
| **RUM** | Real browsers/mobile apps | True user experience (page load, Core Web Vitals) | Datadog RUM, New Relic Browser |
| **APM** | Application agents/SDKs | Code-level bottlenecks, traces, errors | Datadog APM, New Relic, Dynatrace, Elastic APM |

**🔹 Blackbox probe example**
```yaml
- job_name: blackbox
  metrics_path: /probe
  params:
    module: [http_2xx]
  static_configs:
    - targets: ["https://www.example.com"]
```

> **Key takeaway:** Synthetic = *"Is it up?"*, RUM = *"How do real users feel?"*, APM = *"Which code is slow?"*

<br>

---

<br>