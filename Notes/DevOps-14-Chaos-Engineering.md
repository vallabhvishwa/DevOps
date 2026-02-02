# DevOps Engineer's Complete Reference Guide
# Part 14: Chaos Engineering

---

## Table of Contents

1. [Introduction to Chaos Engineering](#1-introduction-to-chaos-engineering)
2. [Chaos Engineering Principles](#2-chaos-engineering-principles)
3. [Chaos Mesh](#3-chaos-mesh)
4. [Litmus Chaos](#4-litmus-chaos)
5. [Azure Chaos Studio](#5-azure-chaos-studio)
6. [Designing Experiments](#6-designing-experiments)
7. [GameDays](#7-gamedays)
8. [Best Practices](#8-best-practices)

---

## 1. Introduction to Chaos Engineering

### 1.1 What is Chaos Engineering?

```
Chaos Engineering Definition:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  "Chaos Engineering is the discipline of experimenting on a    │
│   system in order to build confidence in the system's          │
│   capability to withstand turbulent conditions in production." │
│                                     - Principles of Chaos       │
│                                                                 │
│  Goal: Proactively discover weaknesses before they cause       │
│        real-world failures                                      │
│                                                                 │
│  Why Chaos Engineering?                                         │
│  ├── Distributed systems fail in unpredictable ways            │
│  ├── Testing in isolation doesn't catch all issues             │
│  ├── Builds confidence in system resilience                    │
│  ├── Reduces incident severity and duration                    │
│  └── Forces teams to design for failure                        │
│                                                                 │
│  Common Failure Modes Tested:                                   │
│  ├── Pod/Container crashes                                      │
│  ├── Node failures                                              │
│  ├── Network partitions                                         │
│  ├── Latency injection                                          │
│  ├── Resource exhaustion (CPU, memory, disk)                    │
│  ├── DNS failures                                               │
│  └── Dependency failures                                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Chaos Engineering Principles

```
Chaos Engineering Process:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. DEFINE STEADY STATE                                         │
│     ├── What does "normal" look like?                           │
│     ├── Key metrics: latency, error rate, throughput            │
│     └── Business metrics: orders/min, logins/min                │
│                                                                 │
│  2. HYPOTHESIZE                                                 │
│     ├── "The system will remain stable when..."                 │
│     ├── Define expected behavior under stress                   │
│     └── Predict impact of the experiment                        │
│                                                                 │
│  3. INTRODUCE REAL-WORLD EVENTS                                 │
│     ├── Kill pods/nodes                                         │
│     ├── Add network latency                                     │
│     ├── Simulate AZ failure                                     │
│     └── Exhaust resources                                       │
│                                                                 │
│  4. MEASURE AND OBSERVE                                         │
│     ├── Compare actual vs expected behavior                     │
│     ├── Monitor key SLIs                                        │
│     └── Document observations                                   │
│                                                                 │
│  5. IMPROVE                                                     │
│     ├── Fix weaknesses discovered                               │
│     ├── Update runbooks                                         │
│     ├── Improve monitoring                                      │
│     └── Re-run experiment                                       │
│                                                                 │
│  Key Rules:                                                     │
│  ├── Start small (blast radius)                                 │
│  ├── Have a kill switch                                         │
│  ├── Run in production (carefully)                              │
│  └── Automate experiments                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Chaos Mesh

### 3.1 Installing Chaos Mesh

```bash
# Install via Helm
helm repo add chaos-mesh https://charts.chaos-mesh.org
helm install chaos-mesh chaos-mesh/chaos-mesh \
  --namespace chaos-mesh \
  --create-namespace \
  --set chaosDaemon.runtime=containerd \
  --set chaosDaemon.socketPath=/run/containerd/containerd.sock

# Access dashboard
kubectl port-forward -n chaos-mesh svc/chaos-dashboard 2333:2333
# Open http://localhost:2333
```

### 3.2 Chaos Mesh Experiments

```yaml
# Pod Chaos - Kill pods
apiVersion: chaos-mesh.org/v1alpha1
kind: PodChaos
metadata:
  name: pod-failure
  namespace: chaos-mesh
spec:
  action: pod-failure
  mode: one  # one, all, fixed, fixed-percent, random-max-percent
  selector:
    namespaces:
    - production
    labelSelectors:
      app: myapp
  duration: "30s"
  scheduler:
    cron: "@every 5m"

---
# Network Chaos - Add latency
apiVersion: chaos-mesh.org/v1alpha1
kind: NetworkChaos
metadata:
  name: network-delay
spec:
  action: delay
  mode: all
  selector:
    namespaces:
    - production
    labelSelectors:
      app: api
  delay:
    latency: "100ms"
    jitter: "20ms"
    correlation: "50"
  direction: to
  target:
    selector:
      namespaces:
      - production
      labelSelectors:
        app: database
    mode: all
  duration: "2m"

---
# Network Chaos - Packet loss
apiVersion: chaos-mesh.org/v1alpha1
kind: NetworkChaos
metadata:
  name: network-loss
spec:
  action: loss
  mode: all
  selector:
    labelSelectors:
      app: myapp
  loss:
    loss: "25"
    correlation: "25"
  duration: "1m"

---
# Stress Chaos - CPU stress
apiVersion: chaos-mesh.org/v1alpha1
kind: StressChaos
metadata:
  name: cpu-stress
spec:
  mode: all
  selector:
    labelSelectors:
      app: myapp
  stressors:
    cpu:
      workers: 2
      load: 80  # 80% CPU load
  duration: "5m"

---
# Stress Chaos - Memory stress
apiVersion: chaos-mesh.org/v1alpha1
kind: StressChaos
metadata:
  name: memory-stress
spec:
  mode: one
  selector:
    labelSelectors:
      app: myapp
  stressors:
    memory:
      workers: 2
      size: "500MB"
  duration: "2m"

---
# IO Chaos - Slow disk
apiVersion: chaos-mesh.org/v1alpha1
kind: IOChaos
metadata:
  name: io-latency
spec:
  action: latency
  mode: one
  selector:
    labelSelectors:
      app: database
  volumePath: /data
  path: "*"
  delay: "100ms"
  percent: 50
  duration: "5m"

---
# DNS Chaos
apiVersion: chaos-mesh.org/v1alpha1
kind: DNSChaos
metadata:
  name: dns-error
spec:
  action: error
  mode: all
  selector:
    labelSelectors:
      app: myapp
  patterns:
  - "*.external-api.com"
  duration: "2m"

---
# HTTP Chaos (Istio required)
apiVersion: chaos-mesh.org/v1alpha1
kind: HTTPChaos
metadata:
  name: http-abort
spec:
  mode: all
  selector:
    labelSelectors:
      app: api
  target: Request
  port: 8080
  method: GET
  path: /api/orders
  abort: true
  code: 503
  percent: 50
  duration: "1m"
```

### 3.3 Workflow (Multi-step Experiments)

```yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: Workflow
metadata:
  name: cascading-failure-test
spec:
  entry: start
  templates:
  - name: start
    templateType: Serial
    children:
    - inject-latency
    - wait-and-observe
    - kill-pod
    - verify-recovery
  
  - name: inject-latency
    templateType: NetworkChaos
    deadline: 5m
    networkChaos:
      action: delay
      mode: all
      selector:
        labelSelectors:
          app: database
      delay:
        latency: "200ms"
  
  - name: wait-and-observe
    templateType: Suspend
    deadline: 2m
  
  - name: kill-pod
    templateType: PodChaos
    deadline: 30s
    podChaos:
      action: pod-kill
      mode: one
      selector:
        labelSelectors:
          app: api
  
  - name: verify-recovery
    templateType: Suspend
    deadline: 3m
```

---

## 4. Litmus Chaos

### 4.1 Installing Litmus

```bash
# Install Litmus operator
kubectl apply -f https://litmuschaos.github.io/litmus/3.0.0/litmus-3.0.0.yaml

# Access Litmus portal
kubectl port-forward svc/litmusportal-frontend-service -n litmus 9091:9091

# Install chaos experiments
kubectl apply -f https://hub.litmuschaos.io/api/chaos/3.0.0?file=charts/generic/experiments.yaml
```

### 4.2 Litmus Experiments

```yaml
# ChaosEngine - Run experiment
apiVersion: litmuschaos.io/v1alpha1
kind: ChaosEngine
metadata:
  name: pod-delete-chaos
  namespace: production
spec:
  appinfo:
    appns: production
    applabel: app=myapp
    appkind: deployment
  engineState: active
  chaosServiceAccount: litmus-admin
  experiments:
  - name: pod-delete
    spec:
      components:
        env:
        - name: TOTAL_CHAOS_DURATION
          value: "30"
        - name: CHAOS_INTERVAL
          value: "10"
        - name: FORCE
          value: "false"
        - name: PODS_AFFECTED_PERC
          value: "50"

---
# Container kill experiment
apiVersion: litmuschaos.io/v1alpha1
kind: ChaosEngine
metadata:
  name: container-kill
spec:
  appinfo:
    appns: production
    applabel: app=myapp
    appkind: deployment
  engineState: active
  chaosServiceAccount: litmus-admin
  experiments:
  - name: container-kill
    spec:
      components:
        env:
        - name: TARGET_CONTAINER
          value: "myapp"
        - name: CHAOS_INTERVAL
          value: "10"
        - name: TOTAL_CHAOS_DURATION
          value: "60"

---
# Node drain experiment
apiVersion: litmuschaos.io/v1alpha1
kind: ChaosEngine
metadata:
  name: node-drain
spec:
  appinfo:
    appns: production
    applabel: app=myapp
    appkind: deployment
  engineState: active
  chaosServiceAccount: litmus-admin
  experiments:
  - name: node-drain
    spec:
      components:
        env:
        - name: TOTAL_CHAOS_DURATION
          value: "60"
        - name: APP_NODE
          value: "node-1"
```

---

## 5. Azure Chaos Studio

### 5.1 Setting Up Azure Chaos Studio

```bash
# Enable Chaos Studio on target resource
az rest --method put \
  --url "https://management.azure.com/subscriptions/{subId}/resourceGroups/{rg}/providers/Microsoft.ContainerService/managedClusters/{aksName}/providers/Microsoft.Chaos/targets/Microsoft-AzureKubernetesServiceChaosMesh?api-version=2023-11-01" \
  --body '{"properties":{}}'

# Enable capabilities
az rest --method put \
  --url "https://management.azure.com/.../capabilities/PodChaos-2.1?api-version=2023-11-01" \
  --body '{"properties":{}}'
```

### 5.2 Chaos Experiment Definition

```json
{
  "identity": {
    "type": "SystemAssigned"
  },
  "properties": {
    "selectors": [
      {
        "type": "List",
        "id": "selector1",
        "targets": [
          {
            "type": "ChaosTarget",
            "id": "/subscriptions/.../providers/Microsoft.Chaos/targets/Microsoft-AzureKubernetesServiceChaosMesh"
          }
        ]
      }
    ],
    "steps": [
      {
        "name": "Step 1 - Pod Kill",
        "branches": [
          {
            "name": "Branch 1",
            "actions": [
              {
                "type": "continuous",
                "name": "urn:csci:microsoft:azureKubernetesServiceChaosMesh:podChaos/2.1",
                "duration": "PT5M",
                "parameters": [
                  {
                    "key": "jsonSpec",
                    "value": "{\"action\":\"pod-kill\",\"mode\":\"one\",\"selector\":{\"namespaces\":[\"production\"],\"labelSelectors\":{\"app\":\"myapp\"}}}"
                  }
                ],
                "selectorId": "selector1"
              }
            ]
          }
        ]
      }
    ]
  }
}
```

---

## 6. Designing Experiments

### 6.1 Experiment Template

```markdown
# Chaos Experiment: Database Failover

## Hypothesis
When the primary database pod is killed, the system should:
1. Failover to replica within 30 seconds
2. Maintain availability (no 5xx errors to users)
3. Automatically recover when pod restarts

## Steady State
- API error rate < 0.1%
- P99 latency < 500ms
- Orders processing: 100/min

## Experiment
- Action: Kill primary PostgreSQL pod
- Duration: Observe for 5 minutes
- Scope: Production database (single pod)

## Abort Conditions
- Error rate > 5% for more than 2 minutes
- P99 latency > 5 seconds
- Critical alerts firing

## Rollback Plan
1. Stop chaos experiment (kill switch)
2. If pod not recovering, manually restart
3. If data issues, restore from backup

## Monitoring
- Grafana dashboard: db-failover
- Alert channel: #chaos-experiments
- Oncall: SRE team

## Results
- [ ] Hypothesis confirmed
- [ ] Issues discovered
- [ ] Action items created
```

### 6.2 Blast Radius Control

```yaml
# Progressive experiment scope
# Level 1: Single pod in staging
spec:
  mode: one
  selector:
    namespaces: [staging]
    labelSelectors:
      app: myapp

# Level 2: 25% of pods in staging
spec:
  mode: fixed-percent
  value: "25"
  selector:
    namespaces: [staging]

# Level 3: Single pod in production
spec:
  mode: one
  selector:
    namespaces: [production]

# Level 4: 50% of pods in production
# Only after Level 3 is validated
```

---

## 7. GameDays

```
GameDay Structure:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  PREPARATION (1-2 weeks before):                                │
│  ├── Define scenarios to test                                   │
│  ├── Identify participants (SRE, Dev, Ops)                      │
│  ├── Set up monitoring dashboards                               │
│  ├── Prepare rollback procedures                                │
│  ├── Notify stakeholders                                        │
│  └── Schedule (low-traffic period)                              │
│                                                                 │
│  EXECUTION (GameDay):                                           │
│  ├── Kickoff meeting (30 min)                                   │
│  │   ├── Review objectives                                      │
│  │   ├── Assign roles (facilitator, observer, responder)        │
│  │   └── Confirm kill switch                                    │
│  ├── Run experiments (2-4 hours)                                │
│  │   ├── Execute prepared scenarios                             │
│  │   ├── Observe team response                                  │
│  │   ├── Document everything                                    │
│  │   └── Allow organic troubleshooting                          │
│  └── Debrief (1 hour)                                           │
│      ├── What worked well?                                      │
│      ├── What surprised us?                                     │
│      └── What needs improvement?                                │
│                                                                 │
│  FOLLOW-UP (1 week after):                                      │
│  ├── Document findings                                          │
│  ├── Create action items                                        │
│  ├── Update runbooks                                            │
│  ├── Fix discovered issues                                      │
│  └── Plan next GameDay                                          │
│                                                                 │
│  Example Scenarios:                                             │
│  ├── Primary database failure                                   │
│  ├── AZ unavailable                                             │
│  ├── Certificate expiration                                     │
│  ├── Third-party API outage                                     │
│  ├── DNS failure                                                │
│  └── Secret rotation                                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 8. Best Practices

```
Chaos Engineering Best Practices:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. START SMALL                                                 │
│     ├── Begin in non-production                                 │
│     ├── Single component first                                  │
│     ├── Short duration                                          │
│     └── Gradually increase scope                                │
│                                                                 │
│  2. ALWAYS HAVE A KILL SWITCH                                   │
│     ├── Immediate abort capability                              │
│     ├── Automatic timeout                                       │
│     └── Clear ownership                                         │
│                                                                 │
│  3. MONITOR EVERYTHING                                          │
│     ├── Establish baseline before experiment                    │
│     ├── Real-time dashboards during                             │
│     └── Compare before/during/after                             │
│                                                                 │
│  4. AUTOMATE EXPERIMENTS                                        │
│     ├── Run regularly (CI/CD integration)                       │
│     ├── Automated rollback                                      │
│     └── Consistent execution                                    │
│                                                                 │
│  5. DOCUMENT LEARNINGS                                          │
│     ├── Record all experiments                                  │
│     ├── Track improvements over time                            │
│     └── Share knowledge across teams                            │
│                                                                 │
│  6. CULTURE OF SAFETY                                           │
│     ├── Blameless when things break                             │
│     ├── Celebrate finding weaknesses                            │
│     └── Encourage experimentation                               │
│                                                                 │
│  Anti-patterns to Avoid:                                        │
│  ├── Running chaos without monitoring                           │
│  ├── No hypothesis (random breaking)                            │
│  ├── Too large blast radius initially                           │
│  ├── Not having rollback plan                                   │
│  └── Chaos without stakeholder buy-in                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Summary

Chaos Engineering builds confidence in system resilience:

1. **Purpose**: Proactively discover weaknesses before production incidents
2. **Tools**: Chaos Mesh, Litmus, Azure Chaos Studio
3. **Experiments**: Pod kill, network chaos, stress testing, DNS failures
4. **Process**: Hypothesis → Experiment → Observe → Improve
5. **GameDays**: Scheduled chaos exercises with team participation
6. **Key Principle**: Always have a kill switch and start small

Chaos Engineering transforms how teams think about failure—from reactive incident response to proactive resilience building.

---

**Next Part**: [Part 15: Python & Go for DevOps](./DevOps-Complete-Reference-Guide-Part15-Programming.md)
