# OpenTelemetry — DevOps Guide

> **last_reviewed:** 2026-06-30  
> **Prerequisites:** [Observability](DevOps-07-Observability-Apps.md) | **Project:** [PetClinic](../../Projects/PetClinic.md)

OpenTelemetry (OTel) is the vendor-neutral standard for **traces, metrics, and logs**. It unifies instrumentation and exports to Prometheus, Jaeger, Tempo, Azure Monitor, etc.

---

## 1. Three signals

```
┌─────────────┐   OTLP    ┌──────────────────┐
│ Application │ ────────▶ │ OTel Collector   │
│ (auto/manual│           │ (receive/process │
│  instrument)│           │  /export)        │
└─────────────┘           └────────┬─────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              ▼                    ▼                    ▼
         Prometheus            Tempo/Jaeger         Loki / Azure
         (metrics)              (traces)             (logs)
              │                    │                    │
              └────────────────────┴────────────────────┘
                                   ▼
                              Grafana
                         (correlate by trace_id)
```

---

## 2. Key concepts

| Concept | Description |
|---------|-------------|
| **Span** | Single operation (HTTP request, DB query) |
| **Trace** | Tree of spans for one request |
| **trace_id** | Correlates spans across services |
| **Baggage** | Key-value propagated across services |
| **OTLP** | OpenTelemetry Protocol (gRPC :4317, HTTP :4318) |

---

## 3. Collector configuration

```yaml
# otel-collector-config.yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  batch:
    timeout: 10s
    send_batch_size: 1024
  memory_limiter:
    check_interval: 1s
    limit_mib: 512
  resource:
    attributes:
      - key: deployment.environment
        value: production
        action: upsert

exporters:
  otlp/tempo:
    endpoint: tempo.monitoring.svc:4317
    tls:
      insecure: true
  prometheus:
    endpoint: 0.0.0.0:8889
  logging:
    loglevel: info

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, batch, resource]
      exporters: [otlp/tempo, logging]
    metrics:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [prometheus]
```

Deploy as DaemonSet (node-level) or Deployment (central).

---

## 4. Kubernetes deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: otel-collector
  namespace: monitoring
spec:
  replicas: 2
  selector:
    matchLabels:
      app: otel-collector
  template:
    metadata:
      labels:
        app: otel-collector
    spec:
      containers:
        - name: collector
          image: otel/opentelemetry-collector-contrib:0.96.0
          args: ["--config=/conf/config.yaml"]
          ports:
            - containerPort: 4317
            - containerPort: 4318
            - containerPort: 8889
          volumeMounts:
            - name: config
              mountPath: /conf
      volumes:
        - name: config
          configMap:
            name: otel-collector-config
```

---

## 5. Spring Boot (PetClinic) auto-instrumentation

```yaml
# application.yml or env vars
management:
  tracing:
    sampling:
      probability: 1.0   # dev: 1.0, prod: 0.1
  otlp:
    tracing:
      endpoint: http://otel-collector.monitoring:4318/v1/traces

# Or via env in Helm values:
env:
  OTEL_EXPORTER_OTLP_ENDPOINT: http://otel-collector.monitoring:4318
  OTEL_SERVICE_NAME: customers-service
  OTEL_RESOURCE_ATTRIBUTES: deployment.environment=dev
```

**Java agent (alternative):**

```dockerfile
ADD https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/latest/download/opentelemetry-javaagent.jar /otel.jar
ENV JAVA_TOOL_OPTIONS="-javaagent:/otel.jar"
```

---

## 6. Correlate logs with traces

Structured logs should include `trace_id` and `span_id`:

```json
{
  "timestamp": "2026-06-30T10:00:00Z",
  "level": "ERROR",
  "message": "DB connection failed",
  "trace_id": "abc123...",
  "span_id": "def456..."
}
```

Grafana: jump from log line → trace → related metrics.

---

## 7. Sampling (production)

| Strategy | When |
|----------|------|
| `probability: 1.0` | Dev/staging |
| `probability: 0.1` | Prod — 10% of traces |
| Tail-based sampling | Keep all errors, sample successes (collector contrib) |

High traffic without sampling = expensive storage and collector OOM.

---

## 8. Azure Monitor integration

```yaml
exporters:
  azuremonitor:
    connection_string: "InstrumentationKey=...;IngestionEndpoint=..."
```

Or use **Azure Monitor OpenTelemetry Distro** for AKS — managed agent forwards to Application Insights.

---

## 9. Troubleshooting

| Symptom | Check |
|---------|-------|
| No traces in Tempo | `curl -v http://collector:4318/v1/traces` from app pod |
| Collector OOM | Reduce batch size; add memory_limiter; increase limits |
| Broken trace chain | All services must propagate W3C `traceparent` header |
| High cardinality metrics | Avoid unbounded labels (user IDs in metric labels) |

```bash
# Verify collector receiving data
kubectl logs -n monitoring deploy/otel-collector -f

# Test OTLP HTTP from debug pod
curl -X POST http://otel-collector.monitoring:4318/v1/traces \
  -H 'Content-Type: application/json' -d '{}'
```

---

## Related

- [Observability](DevOps-07-Observability-Apps.md) — Prometheus, Grafana, logging
- [PetClinic Phase 5](../../Projects/PetClinic.md)
- [Kubernetes Troubleshooting](../../Troubleshooting/Kubernetes/02-Kubernetes-Troubleshooting.md)
