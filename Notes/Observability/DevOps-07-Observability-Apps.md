# The Complete DevOps Engineer's Reference Guide
## Part 7: Observability, Applications & Troubleshooting

---

# Chapter 21: Observability - The Three Pillars

## 21.1 Understanding Observability

```
Observability Pillars:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐       │
│  │     METRICS     │ │      LOGS       │ │     TRACES      │       │
│  │                 │ │                 │ │                 │       │
│  │  Numeric data   │ │  Text records   │ │ Request flow    │       │
│  │  over time      │ │  of events      │ │ across services │       │
│  │                 │ │                 │ │                 │       │
│  │  Examples:      │ │  Examples:      │ │  Examples:      │       │
│  │  - CPU usage    │ │  - Error msgs   │ │  - API calls    │       │
│  │  - Latency      │ │  - Access logs  │ │  - DB queries   │       │
│  │  - Error rate   │ │  - Audit logs   │ │  - HTTP requests│       │
│  │                 │ │                 │ │                 │       │
│  │  Tools:         │ │  Tools:         │ │  Tools:         │       │
│  │  - Prometheus   │ │  - Loki         │ │  - Jaeger       │       │
│  │  - Grafana      │ │  - ELK Stack    │ │  - Zipkin       │       │
│  │  - Azure Monitor│ │  - Splunk       │ │  - App Insights │       │
│  │                 │ │  - Azure LA     │ │  - Tempo        │       │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘       │
│                              │                                      │
│                              ▼                                      │
│                    ┌─────────────────┐                             │
│                    │   Correlation   │                             │
│                    │                 │                             │
│                    │  Link all three │                             │
│                    │  with trace IDs │                             │
│                    │  and timestamps │                             │
│                    └─────────────────┘                             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 21.2 Prometheus and Metrics

### Prometheus Architecture

```
Prometheus Architecture:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  Targets (Applications, Kubernetes, etc.)                           │
│    │                                                                │
│    │ HTTP /metrics endpoint                                         │
│    ▼                                                                │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                     Prometheus Server                         │  │
│  │                                                              │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │  │
│  │  │  Retrieval  │  │    TSDB     │  │    HTTP     │          │  │
│  │  │  (Scraping) │─▶│  (Storage)  │◀─│   Server    │          │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘          │  │
│  │                          │               │                   │  │
│  │  ┌─────────────┐        │               │                   │  │
│  │  │   Service   │        │               │                   │  │
│  │  │  Discovery  │        │               │                   │  │
│  │  └─────────────┘        │               │                   │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                             │               │                       │
│                             ▼               ▼                       │
│                    ┌──────────────┐  ┌──────────────┐              │
│                    │ Alertmanager │  │   Grafana    │              │
│                    │              │  │              │              │
│                    │  Routing     │  │ Visualization│              │
│                    │  Grouping    │  │  Dashboards  │              │
│                    │  Notifications│  │              │              │
│                    └──────────────┘  └──────────────┘              │
│                           │                                         │
│                           ▼                                         │
│                    ┌──────────────┐                                │
│                    │ Email/Slack/ │                                │
│                    │ PagerDuty    │                                │
│                    └──────────────┘                                │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### PromQL (Prometheus Query Language)

```promql
# Basic Queries
──────────────────────────────────────────────────────────────────────

# Instant vector - single value per time series
http_requests_total

# Range vector - values over time range
http_requests_total[5m]

# Label filtering
http_requests_total{job="api-server"}
http_requests_total{method="GET", status="200"}
http_requests_total{status=~"2.."}           # Regex match
http_requests_total{status!~"5.."}           # Regex not match
http_requests_total{status!="500"}           # Not equal

# Rate and Increase
──────────────────────────────────────────────────────────────────────

# Per-second rate over 5 minutes (for counters)
rate(http_requests_total[5m])

# Increase over 1 hour
increase(http_requests_total[1h])

# irate - instant rate based on last 2 data points (more volatile)
irate(http_requests_total[5m])

# Aggregation
──────────────────────────────────────────────────────────────────────

# Sum across all instances
sum(rate(http_requests_total[5m]))

# Sum by label
sum by (method) (rate(http_requests_total[5m]))
sum by (method, status) (rate(http_requests_total[5m]))

# Average
avg(node_memory_MemFree_bytes)
avg by (instance) (rate(node_cpu_seconds_total[5m]))

# Count
count(up)
count by (job) (up)

# Min/Max
min(node_memory_MemFree_bytes)
max(node_memory_MemFree_bytes)

# Quantiles
quantile(0.95, http_request_duration_seconds)

# Top/Bottom K
topk(5, rate(http_requests_total[5m]))
bottomk(5, rate(http_requests_total[5m]))

# Functions
──────────────────────────────────────────────────────────────────────

# Histogram quantiles (percentiles)
histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))
histogram_quantile(0.50, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))

# Absent (alert if metric missing)
absent(up{job="api-server"})

# Changes (count changes in gauge)
changes(process_start_time_seconds[1h])

# Delta (difference in gauge)
delta(cpu_temperature_celsius[1h])

# Predict linear (predict value N seconds from now)
predict_linear(node_filesystem_free_bytes[1h], 4*3600)

# Label manipulation
label_replace(up, "short_instance", "$1", "instance", "(.*):.+")

# Common Patterns
──────────────────────────────────────────────────────────────────────

# Error rate (percentage)
sum(rate(http_requests_total{status=~"5.."}[5m])) 
/ 
sum(rate(http_requests_total[5m])) 
* 100

# Latency percentiles
histogram_quantile(0.99, 
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
)

# Container CPU usage
sum(rate(container_cpu_usage_seconds_total{container!=""}[5m])) by (pod)

# Container memory usage
sum(container_memory_working_set_bytes{container!=""}) by (pod)

# Pod restart count
sum(kube_pod_container_status_restarts_total) by (pod, namespace)

# Available replicas
kube_deployment_status_replicas_available{namespace="production"}

# Node CPU utilization
100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Node memory utilization
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Disk usage percentage
(1 - (node_filesystem_avail_bytes / node_filesystem_size_bytes)) * 100

# Request rate by endpoint
sum by (path) (rate(http_requests_total[5m]))

# Alerting expression (error rate > 5%)
sum(rate(http_requests_total{status=~"5.."}[5m])) 
/ 
sum(rate(http_requests_total[5m])) 
> 0.05
```

### Prometheus Configuration

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: 'production'
    env: 'prod'

# Alerting
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

# Rules
rule_files:
  - "alerts/*.yml"
  - "recording_rules/*.yml"

# Scrape configs
scrape_configs:
  # Prometheus itself
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Node exporters
  - job_name: 'node'
    static_configs:
      - targets: 
        - 'node1:9100'
        - 'node2:9100'

  # Kubernetes pods with prometheus.io annotations
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      # Only scrape pods with prometheus.io/scrape=true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      # Get path from annotation
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      # Get port from annotation
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
      # Add pod labels
      - action: labelmap
        regex: __meta_kubernetes_pod_label_(.+)
      # Add namespace label
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: namespace
      # Add pod name label
      - source_labels: [__meta_kubernetes_pod_name]
        action: replace
        target_label: pod
```

### Recording Rules

```yaml
# recording_rules.yml
groups:
  - name: example
    interval: 30s
    rules:
      # Pre-calculate expensive queries
      - record: job:http_requests_total:rate5m
        expr: sum by (job) (rate(http_requests_total[5m]))
      
      - record: job:http_request_latency_seconds:p99
        expr: histogram_quantile(0.99, sum by (le, job) (rate(http_request_duration_seconds_bucket[5m])))
      
      - record: instance:node_cpu:ratio
        expr: 1 - avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m]))
```

### Alert Rules

```yaml
# alerts.yml
groups:
  - name: application
    rules:
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m])) 
          / 
          sum(rate(http_requests_total[5m])) 
          > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value | humanizePercentage }}"

      - alert: HighLatency
        expr: |
          histogram_quantile(0.99, 
            sum by (le) (rate(http_request_duration_seconds_bucket[5m]))
          ) > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High latency detected"
          description: "99th percentile latency is {{ $value }}s"

  - name: kubernetes
    rules:
      - alert: PodCrashLooping
        expr: |
          increase(kube_pod_container_status_restarts_total[1h]) > 5
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Pod {{ $labels.pod }} is crash looping"
          description: "Pod has restarted {{ $value }} times in the last hour"

      - alert: PodNotReady
        expr: |
          kube_pod_status_ready{condition="true"} == 0
        for: 15m
        labels:
          severity: warning
        annotations:
          summary: "Pod {{ $labels.pod }} not ready"

      - alert: NodeHighCPU
        expr: |
          instance:node_cpu:ratio > 0.8
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High CPU on {{ $labels.instance }}"
          description: "CPU usage is {{ $value | humanizePercentage }}"

      - alert: NodeHighMemory
        expr: |
          (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) > 0.9
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: "High memory on {{ $labels.instance }}"
          description: "Memory usage is {{ $value | humanizePercentage }}"

      - alert: DiskSpaceLow
        expr: |
          (1 - (node_filesystem_avail_bytes{fstype!~"tmpfs|overlay"} / node_filesystem_size_bytes)) > 0.85
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Low disk space on {{ $labels.instance }}"
          description: "Disk usage is {{ $value | humanizePercentage }}"
```

## 21.3 Logging

### Log Aggregation Architecture

```
Logging Architecture:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  Applications / Containers                                          │
│  │ (stdout/stderr, log files)                                       │
│  │                                                                  │
│  ▼                                                                  │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │              Log Collectors (DaemonSet)                       │  │
│  │                                                              │  │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐              │  │
│  │  │  Fluent Bit│  │  Fluentd   │  │   Vector   │              │  │
│  │  │ (lightweight)│ │  (plugins) │  │  (modern)  │              │  │
│  │  └────────────┘  └────────────┘  └────────────┘              │  │
│  │                                                              │  │
│  │  - Tail container logs from /var/log/containers/             │  │
│  │  - Parse, filter, enrich                                     │  │
│  │  - Forward to storage                                        │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                              │                                      │
│              ┌───────────────┼───────────────┐                      │
│              ▼               ▼               ▼                      │
│  ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐    │
│  │  Elasticsearch   │ │      Loki        │ │ Azure Log        │    │
│  │                  │ │                  │ │ Analytics        │    │
│  │  - Full-text     │ │  - Label-based   │ │                  │    │
│  │  - Powerful      │ │  - Lightweight   │ │  - KQL queries   │    │
│  │  - Resource heavy│ │  - Cost-effective│ │  - Azure native  │    │
│  └──────────────────┘ └──────────────────┘ └──────────────────┘    │
│              │               │               │                      │
│              ▼               ▼               ▼                      │
│  ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐    │
│  │     Kibana       │ │     Grafana      │ │  Azure Portal    │    │
│  │                  │ │                  │ │                  │    │
│  │  Visualization   │ │  Visualization   │ │  Workbooks       │    │
│  └──────────────────┘ └──────────────────┘ └──────────────────┘    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Fluent Bit Configuration

```yaml
# fluent-bit.conf as ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluent-bit-config
  namespace: logging
data:
  fluent-bit.conf: |
    [SERVICE]
        Flush         5
        Log_Level     info
        Daemon        off
        Parsers_File  parsers.conf
        HTTP_Server   On
        HTTP_Listen   0.0.0.0
        HTTP_Port     2020

    [INPUT]
        Name              tail
        Tag               kube.*
        Path              /var/log/containers/*.log
        Parser            docker
        DB                /var/log/flb_kube.db
        Mem_Buf_Limit     50MB
        Skip_Long_Lines   On
        Refresh_Interval  10

    [FILTER]
        Name                kubernetes
        Match               kube.*
        Kube_URL            https://kubernetes.default.svc:443
        Kube_CA_File        /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        Kube_Token_File     /var/run/secrets/kubernetes.io/serviceaccount/token
        Merge_Log           On
        K8S-Logging.Parser  On
        K8S-Logging.Exclude On

    [FILTER]
        Name          modify
        Match         *
        Add           cluster production
        Add           environment prod

    [OUTPUT]
        Name            loki
        Match           *
        Host            loki.logging.svc
        Port            3100
        Labels          job=fluent-bit, cluster=production
        Label_Keys      $kubernetes['namespace_name'],$kubernetes['pod_name']
        Remove_Keys     kubernetes,stream

  parsers.conf: |
    [PARSER]
        Name        docker
        Format      json
        Time_Key    time
        Time_Format %Y-%m-%dT%H:%M:%S.%L
        Time_Keep   On

    [PARSER]
        Name        json
        Format      json
        Time_Key    timestamp
        Time_Format %Y-%m-%dT%H:%M:%S.%L
```

### Azure Log Analytics (KQL Queries)

```kusto
// Container logs
ContainerLogV2
| where TimeGenerated > ago(1h)
| where ContainerName == "myapp"
| project TimeGenerated, LogEntry, PodName
| order by TimeGenerated desc
| take 100

// Error logs
ContainerLogV2
| where TimeGenerated > ago(24h)
| where LogEntry contains "error" or LogEntry contains "exception"
| summarize ErrorCount=count() by bin(TimeGenerated, 1h), ContainerName
| render timechart

// Pod restarts
KubePodInventory
| where TimeGenerated > ago(24h)
| summarize RestartCount=sum(PodRestartCount) by Name, Namespace, bin(TimeGenerated, 1h)
| where RestartCount > 0
| order by RestartCount desc

// Node CPU usage
Perf
| where TimeGenerated > ago(1h)
| where ObjectName == "K8SNode"
| where CounterName == "cpuUsageNanoCores"
| summarize AvgCPU=avg(CounterValue) by bin(TimeGenerated, 5m), Computer
| render timechart

// Memory pressure
Perf
| where TimeGenerated > ago(1h)
| where ObjectName == "K8SNode"
| where CounterName == "memoryWorkingSetBytes"
| extend MemoryGB = CounterValue / 1073741824
| summarize AvgMemory=avg(MemoryGB) by bin(TimeGenerated, 5m), Computer
| render timechart

// Failed deployments
KubeEvents
| where TimeGenerated > ago(24h)
| where Reason in ("Failed", "FailedCreate", "FailedScheduling")
| project TimeGenerated, Name, Namespace, Reason, Message

// Container resource consumption
let threshold = 80;
Perf
| where ObjectName == "K8SContainer"
| where CounterName == "cpuUsageNanoCores" or CounterName == "memoryWorkingSetBytes"
| summarize 
    AvgCPU = avgif(CounterValue, CounterName == "cpuUsageNanoCores"),
    AvgMemory = avgif(CounterValue, CounterName == "memoryWorkingSetBytes")
  by InstanceName
| extend MemoryMB = AvgMemory / 1048576
| order by AvgCPU desc

// Slow API requests
AppRequests
| where TimeGenerated > ago(1h)
| where DurationMs > 1000
| summarize SlowRequests=count(), AvgDuration=avg(DurationMs) by Name
| order by SlowRequests desc
```

## 21.4 Distributed Tracing

### OpenTelemetry Integration

```yaml
# OpenTelemetry Collector deployment
apiVersion: v1
kind: ConfigMap
metadata:
  name: otel-collector-config
data:
  config.yaml: |
    receivers:
      otlp:
        protocols:
          grpc:
            endpoint: 0.0.0.0:4317
          http:
            endpoint: 0.0.0.0:4318
      jaeger:
        protocols:
          grpc:
            endpoint: 0.0.0.0:14250
          thrift_http:
            endpoint: 0.0.0.0:14268
      prometheus:
        config:
          scrape_configs:
            - job_name: 'otel-collector'
              scrape_interval: 10s
              static_configs:
                - targets: ['localhost:8888']
    
    processors:
      batch:
        timeout: 10s
        send_batch_size: 1024
      memory_limiter:
        check_interval: 1s
        limit_mib: 1000
        spike_limit_mib: 200
      resource:
        attributes:
          - key: environment
            value: production
            action: upsert
    
    exporters:
      otlp:
        endpoint: "tempo.monitoring:4317"
        tls:
          insecure: true
      prometheus:
        endpoint: "0.0.0.0:8889"
      logging:
        loglevel: info
    
    service:
      pipelines:
        traces:
          receivers: [otlp, jaeger]
          processors: [memory_limiter, batch]
          exporters: [otlp, logging]
        metrics:
          receivers: [otlp, prometheus]
          processors: [memory_limiter, batch]
          exporters: [prometheus]
```

---

# Chapter 22: Application Knowledge for DevOps

## 22.1 Java Applications

### JVM Memory Model

```
JVM Memory Structure:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                      Heap Memory                             │   │
│  │  ┌───────────────────┐ ┌───────────────────────────────────┐│   │
│  │  │   Young Generation │ │         Old Generation           ││   │
│  │  │                    │ │                                   ││   │
│  │  │ ┌────┐ ┌─────────┐ │ │  Long-lived objects              ││   │
│  │  │ │Eden│ │Survivor │ │ │  Promoted from young gen         ││   │
│  │  │ │    │ │ S0 │ S1 │ │ │                                   ││   │
│  │  │ └────┘ └────┴────┘ │ │                                   ││   │
│  │  │                    │ │                                   ││   │
│  │  │ New objects here   │ │                                   ││   │
│  │  └───────────────────┘ └───────────────────────────────────┘│   │
│  │                                                             │   │
│  │  Configured via: -Xms (initial), -Xmx (maximum)             │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    Non-Heap Memory                           │   │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────────┐│   │
│  │  │  Metaspace  │ │ Code Cache  │ │    Thread Stacks        ││   │
│  │  │             │ │             │ │                         ││   │
│  │  │ Class meta- │ │ JIT compiled│ │ One per thread          ││   │
│  │  │ data        │ │ code        │ │ -Xss per stack size     ││   │
│  │  └─────────────┘ └─────────────┘ └─────────────────────────┘│   │
│  │                                                             │   │
│  │  -XX:MetaspaceSize, -XX:MaxMetaspaceSize                    │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### JVM Container Settings

```bash
# Container-aware JVM settings (Java 11+)
java \
  -XX:+UseContainerSupport \
  -XX:MaxRAMPercentage=75.0 \
  -XX:InitialRAMPercentage=50.0 \
  -XX:MinRAMPercentage=25.0 \
  -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=200 \
  -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=/tmp/heapdump.hprof \
  -Djava.security.egd=file:/dev/./urandom \
  -jar app.jar

# Memory calculation:
# Container limit: 2GB
# MaxRAMPercentage=75% → Heap max = 1.5GB
# Leave 25% for non-heap (metaspace, native memory, etc.)

# For CPU:
# JVM automatically detects container CPU limits
# Uses for thread pool sizing, GC threads, etc.
```

### Java Troubleshooting Commands

```bash
# Find Java processes
jps -l                                # List Java processes
jps -lvm                              # With args and VM options

# Thread dump (for deadlocks, hangs)
jstack <pid>                          # Thread dump
jstack -l <pid>                       # With lock info
jcmd <pid> Thread.print               # Alternative
kill -3 <pid>                         # Sends to stdout/logs

# Heap dump (for memory issues)
jmap -dump:format=b,file=heap.hprof <pid>
jcmd <pid> GC.heap_dump /tmp/heap.hprof

# GC information
jstat -gc <pid> 1000                  # GC stats every 1s
jstat -gcutil <pid> 1000              # GC percentages
jcmd <pid> GC.heap_info               # Heap info
jcmd <pid> GC.run                     # Trigger GC

# Class loading
jcmd <pid> VM.classloader_stats
jcmd <pid> GC.class_stats

# VM flags
jcmd <pid> VM.flags                   # All flags
jcmd <pid> VM.command_line            # Command line

# Flight Recorder (profiling)
jcmd <pid> JFR.start duration=60s filename=/tmp/recording.jfr
jcmd <pid> JFR.dump filename=/tmp/recording.jfr
jcmd <pid> JFR.stop

# In Kubernetes
kubectl exec -it <pod> -- jcmd 1 Thread.print
kubectl exec -it <pod> -- jcmd 1 GC.heap_info
kubectl exec -it <pod> -- jcmd 1 VM.flags
```

### Spring Boot Actuator

```yaml
# application.yml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus,env,loggers
      base-path: /actuator
  endpoint:
    health:
      show-details: always
      probes:
        enabled: true
  health:
    livenessState:
      enabled: true
    readinessState:
      enabled: true
  metrics:
    export:
      prometheus:
        enabled: true

# Kubernetes probes using actuator
# livenessProbe:
#   httpGet:
#     path: /actuator/health/liveness
#     port: 8080
# readinessProbe:
#   httpGet:
#     path: /actuator/health/readiness
#     port: 8080
```

## 22.2 Node.js Applications

### Node.js Memory and Event Loop

```
Node.js Architecture:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                     V8 JavaScript Engine                     │   │
│  │                                                             │   │
│  │  ┌─────────────────────┐  ┌─────────────────────────────┐  │   │
│  │  │      Call Stack     │  │          Heap               │  │   │
│  │  │                     │  │                             │  │   │
│  │  │  Function execution │  │  Object allocation         │  │   │
│  │  │  Single-threaded    │  │  Garbage collected         │  │   │
│  │  │                     │  │                             │  │   │
│  │  │  --stack-size       │  │  --max-old-space-size      │  │   │
│  │  └─────────────────────┘  └─────────────────────────────┘  │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│                              ▼                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                       Event Loop                             │   │
│  │                                                             │   │
│  │  ┌─────────────┐                                           │   │
│  │  │   timers    │ setTimeout, setInterval                   │   │
│  │  └──────┬──────┘                                           │   │
│  │         ▼                                                   │   │
│  │  ┌─────────────┐                                           │   │
│  │  │ pending I/O │ I/O callbacks                             │   │
│  │  └──────┬──────┘                                           │   │
│  │         ▼                                                   │   │
│  │  ┌─────────────┐                                           │   │
│  │  │    poll     │ Retrieve new I/O events                   │   │
│  │  └──────┬──────┘                                           │   │
│  │         ▼                                                   │   │
│  │  ┌─────────────┐                                           │   │
│  │  │    check    │ setImmediate                              │   │
│  │  └──────┬──────┘                                           │   │
│  │         ▼                                                   │   │
│  │  ┌─────────────┐                                           │   │
│  │  │close events │ socket.on('close')                        │   │
│  │  └─────────────┘                                           │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                       libuv                                  │   │
│  │                                                             │   │
│  │  Thread pool for:                                           │   │
│  │  - File I/O                                                 │   │
│  │  - DNS lookups                                              │   │
│  │  - Crypto operations                                        │   │
│  │                                                             │   │
│  │  UV_THREADPOOL_SIZE (default: 4)                           │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Node.js Container Settings

```bash
# Memory settings
node --max-old-space-size=1024 app.js  # 1GB heap limit

# In containers, calculate based on limit:
# Container: 2GB
# Heap: ~1.5GB (leave room for other memory)
node --max-old-space-size=1536 app.js

# For debugging
node --inspect=0.0.0.0:9229 app.js     # Remote debugging
node --inspect-brk app.js               # Break on first line

# Performance
node --enable-source-maps app.js        # Better stack traces
```

### Node.js Health Checks

```javascript
// Express.js health check endpoints
const express = require('express');
const app = express();

// Readiness probe - can we handle requests?
app.get('/health/ready', async (req, res) => {
  try {
    // Check database connection
    await db.query('SELECT 1');
    // Check cache connection
    await cache.ping();
    
    res.status(200).json({ status: 'ready' });
  } catch (error) {
    res.status(503).json({ status: 'not ready', error: error.message });
  }
});

// Liveness probe - is the process alive?
app.get('/health/live', (req, res) => {
  res.status(200).json({ status: 'alive' });
});

// Detailed health with metrics
app.get('/health', (req, res) => {
  const memUsage = process.memoryUsage();
  res.json({
    status: 'healthy',
    uptime: process.uptime(),
    memory: {
      heapUsed: Math.round(memUsage.heapUsed / 1024 / 1024) + 'MB',
      heapTotal: Math.round(memUsage.heapTotal / 1024 / 1024) + 'MB',
      rss: Math.round(memUsage.rss / 1024 / 1024) + 'MB'
    },
    pid: process.pid
  });
});

// Graceful shutdown
process.on('SIGTERM', () => {
  console.log('SIGTERM received, shutting down gracefully');
  
  server.close(() => {
    console.log('HTTP server closed');
    
    // Close database connections
    db.end(() => {
      console.log('Database connection closed');
      process.exit(0);
    });
  });
  
  // Force shutdown after 30 seconds
  setTimeout(() => {
    console.error('Forced shutdown');
    process.exit(1);
  }, 30000);
});
```

---

# Chapter 23: Troubleshooting Guide

## 23.1 Kubernetes Troubleshooting Flowchart

```
Pod Troubleshooting Flowchart:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  kubectl get pods                                                   │
│         │                                                           │
│         ▼                                                           │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              What is the Pod status?                         │   │
│  └─────────────────────────────────────────────────────────────┘   │
│         │                                                           │
│    ┌────┼────────┬────────────┬────────────┬──────────────┐        │
│    │    │        │            │            │              │        │
│    ▼    ▼        ▼            ▼            ▼              ▼        │
│ Pending Running  Error    CrashLoop   ImagePull     Terminating    │
│    │    │        │        BackOff     BackOff           │          │
│    │    │        │            │            │            │          │
│    ▼    │        ▼            ▼            ▼            ▼          │
│         │                                                          │
│ ┌───────┴───────────────────────────────────────────────────────┐  │
│ │                                                               │  │
│ │ PENDING:                                                      │  │
│ │ • kubectl describe pod <pod> - Check Events section          │  │
│ │ • Insufficient resources? Check node capacity                │  │
│ │ • Node selector/affinity not matching? Check node labels     │  │
│ │ • Taint preventing scheduling? Check tolerations             │  │
│ │ • PVC not bound? Check PV/StorageClass                       │  │
│ │                                                               │  │
│ │ RUNNING but unhealthy:                                        │  │
│ │ • kubectl logs <pod> - Check application logs                │  │
│ │ • kubectl describe pod <pod> - Check probe failures          │  │
│ │ • kubectl exec -it <pod> -- sh - Debug inside container      │  │
│ │                                                               │  │
│ │ ERROR / CRASHLOOPBACKOFF:                                     │  │
│ │ • kubectl logs <pod> --previous - Logs from crashed container│  │
│ │ • kubectl describe pod <pod> - Check exit code, events       │  │
│ │ • Exit code 137: OOMKilled (increase memory limit)           │  │
│ │ • Exit code 1: Application error (check app logs)            │  │
│ │                                                               │  │
│ │ IMAGEPULLBACKOFF:                                             │  │
│ │ • Check image name and tag                                    │  │
│ │ • Check imagePullSecrets                                      │  │
│ │ • Check registry authentication                               │  │
│ │ • kubectl get secrets - Verify secret exists                  │  │
│ │                                                               │  │
│ │ TERMINATING (stuck):                                          │  │
│ │ • Check finalizers: kubectl get pod <pod> -o yaml            │  │
│ │ • Force delete: kubectl delete pod <pod> --force --grace-period=0 │  │
│ │                                                               │  │
│ └───────────────────────────────────────────────────────────────┘  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 23.2 Common Issues and Solutions

### Pod Issues

```bash
# Pod stuck in Pending
kubectl describe pod <pod>            # Look at Events
kubectl get events --field-selector reason=FailedScheduling
kubectl describe nodes | grep -A5 "Allocated resources"  # Check capacity
kubectl get pvc                       # Check PVC binding

# Pod CrashLoopBackOff
kubectl logs <pod> --previous         # Previous container logs
kubectl describe pod <pod>            # Check exit code, events
kubectl get pod <pod> -o yaml | grep -A5 "lastState"  # Exit code

# Exit codes:
# 0   : Success
# 1   : Application error
# 137 : SIGKILL (OOMKilled or forced termination)
# 139 : SIGSEGV (segmentation fault)
# 143 : SIGTERM (graceful shutdown)

# OOMKilled fix
kubectl get pod <pod> -o yaml | grep -A3 "resources"
# Increase memory limit in deployment

# ImagePullBackOff
kubectl describe pod <pod> | grep -A10 "Events"
# Check image name
kubectl get pod <pod> -o jsonpath='{.spec.containers[*].image}'
# Check pull secret
kubectl get secrets
kubectl get pod <pod> -o jsonpath='{.spec.imagePullSecrets}'

# CreateContainerConfigError
kubectl describe pod <pod>
# Usually: missing configmap, secret, or PVC
kubectl get configmaps
kubectl get secrets
kubectl get pvc
```

### Service and Networking Issues

```bash
# Service not accessible
kubectl get svc <service>             # Check service exists
kubectl get endpoints <service>       # Check endpoints
kubectl describe svc <service>        # Check selector

# No endpoints
kubectl get pods -l <selector>        # Check pods match selector
kubectl get pods -l <selector> -o wide  # Check pods are Ready

# DNS issues
kubectl run debug --rm -it --image=busybox -- nslookup <service>
kubectl run debug --rm -it --image=busybox -- nslookup kubernetes
kubectl get pods -n kube-system -l k8s-app=kube-dns  # Check CoreDNS
kubectl logs -n kube-system -l k8s-app=kube-dns

# Network connectivity
kubectl run debug --rm -it --image=nicolaka/netshoot -- bash
# Inside: ping, curl, nslookup, traceroute, tcpdump

# Network Policy blocking
kubectl get networkpolicy
kubectl describe networkpolicy <name>

# Ingress issues
kubectl get ingress
kubectl describe ingress <name>
kubectl logs -n <ingress-ns> -l <ingress-controller-label>
```

### Node Issues

```bash
# Node NotReady
kubectl describe node <node>          # Check Conditions
kubectl get events --field-selector involvedObject.name=<node>

# Common conditions:
# MemoryPressure: High memory usage
# DiskPressure: Low disk space
# PIDPressure: Too many processes
# NetworkUnavailable: Network not configured

# Node resource pressure
kubectl top nodes
kubectl describe node <node> | grep -A10 "Allocated resources"

# Drain node for maintenance
kubectl cordon <node>                 # Mark unschedulable
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
# After maintenance
kubectl uncordon <node>
```

### Storage Issues

```bash
# PVC stuck in Pending
kubectl describe pvc <name>           # Check events
kubectl get storageclass              # Check storage class exists
kubectl get pv                        # Check PV availability

# Volume not mounting
kubectl describe pod <pod>            # Check volume mount errors
kubectl get events --field-selector reason=FailedMount

# Permission issues
kubectl exec -it <pod> -- ls -la /path/to/mount
kubectl exec -it <pod> -- id         # Check user ID
# Fix: set fsGroup in securityContext
```

## 23.3 Debug Commands Reference

```bash
# Quick debug pod
kubectl run debug --rm -it --image=nicolaka/netshoot -- bash
kubectl run debug --rm -it --image=busybox -- sh
kubectl run debug --rm -it --image=curlimages/curl -- sh

# Debug existing pod
kubectl debug <pod> -it --image=busybox
kubectl debug <pod> -it --copy-to=debug-pod --container=debug

# Debug node
kubectl debug node/<node> -it --image=ubuntu

# Check cluster health
kubectl get componentstatuses         # Deprecated but sometimes useful
kubectl cluster-info
kubectl get nodes
kubectl get pods -A | grep -v Running

# Events
kubectl get events --sort-by='.lastTimestamp'
kubectl get events -A --sort-by='.lastTimestamp'
kubectl get events --field-selector type=Warning

# Resource usage
kubectl top nodes
kubectl top pods --all-namespaces
kubectl top pods --containers

# Logs
kubectl logs <pod>
kubectl logs <pod> -c <container>
kubectl logs <pod> --all-containers
kubectl logs -l app=myapp --all-containers
kubectl logs <pod> --since=1h
kubectl logs <pod> -f                  # Follow

# API server
kubectl get --raw /healthz
kubectl get --raw /metrics
kubectl auth can-i create pods
kubectl auth can-i --list

# Describe everything
kubectl describe all
kubectl describe pods,svc,deploy,ingress
```

## 23.4 Performance Troubleshooting

```bash
# High CPU
kubectl top pods --sort-by=cpu
kubectl describe pod <high-cpu-pod>
# Java: jstack for thread dump
# Node: --inspect for profiling

# High Memory
kubectl top pods --sort-by=memory
kubectl get pod <pod> -o yaml | grep -A10 resources
# Check for memory leaks

# Slow responses
kubectl logs <pod> | grep -i slow
kubectl logs <pod> | grep -E "[0-9]+ ?ms"
# Check network latency
kubectl exec <pod> -- ping <other-pod-ip>

# Throttling
kubectl describe pod <pod> | grep -i throttl
# CFS throttling - increase CPU limit

# Disk I/O
kubectl exec <pod> -- iostat -x 1 5
kubectl exec <pod> -- df -h
```

---

This concludes Part 7 covering Observability, Applications, and Troubleshooting.

**The Complete Reference Guide includes:**
1. Part 1: Linux Mastery
2. Part 2: Linux Advanced
3. Part 3: Networking Deep Dive
4. Part 4: Kubernetes Deep Dive
5. Part 5: AKS & Azure Services
6. Part 6: Jenkins CI/CD
7. Part 7: Observability, Applications & Troubleshooting

For additional topics (IaC with Terraform/Helm, Security, GitOps), refer to the roadmap document or request additional parts.
