# DevOps Engineer's Complete Reference Guide
# Part 10: Site Reliability Engineering (SRE) Practices

---

## Table of Contents

1. [Introduction to SRE](#1-introduction-to-sre)
2. [SLIs, SLOs, and SLAs](#2-slis-slos-and-slas)
3. [Error Budgets](#3-error-budgets)
4. [Toil Reduction](#4-toil-reduction)
5. [Incident Management](#5-incident-management)
6. [On-Call Best Practices](#6-on-call-best-practices)
7. [Postmortems](#7-postmortems)
8. [Reliability Patterns](#8-reliability-patterns)
9. [Change Management](#9-change-management)
10. [SRE Tools and Automation](#10-sre-tools-and-automation)

---

## 1. Introduction to SRE

### 1.1 What is SRE?

Site Reliability Engineering (SRE) is a discipline that applies software engineering principles to infrastructure and operations problems. Created at Google, SRE treats operations as a software problem.

```
SRE Core Principles:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. Embrace Risk                                                │
│     └── 100% reliability is wrong target (too expensive)        │
│     └── Define acceptable risk levels (SLOs)                    │
│                                                                 │
│  2. Service Level Objectives (SLOs)                             │
│     └── Quantifiable targets for service reliability            │
│     └── Balance reliability with development velocity           │
│                                                                 │
│  3. Error Budgets                                               │
│     └── Acceptable amount of unreliability                      │
│     └── When exhausted, focus shifts to reliability             │
│                                                                 │
│  4. Eliminate Toil                                              │
│     └── Automate repetitive, manual work                        │
│     └── SREs spend ≤50% time on toil                           │
│                                                                 │
│  5. Monitoring and Alerting                                     │
│     └── Symptoms, not causes                                    │
│     └── Actionable alerts only                                  │
│                                                                 │
│  6. Release Engineering                                         │
│     └── Frequent, small, safe releases                          │
│     └── Canary deployments, feature flags                       │
│                                                                 │
│  7. Simplicity                                                  │
│     └── Simple systems are more reliable                        │
│     └── Remove unnecessary complexity                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 SRE vs DevOps

```
SRE and DevOps Relationship:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  DevOps: Philosophy and culture                                 │
│  SRE: Specific implementation of DevOps principles              │
│                                                                 │
│  "SRE is what happens when you ask a software engineer to       │
│   design an operations function" - Ben Treynor Sloss, Google    │
│                                                                 │
│  DevOps Principle        SRE Implementation                     │
│  ─────────────────────────────────────────────────────────────  │
│  Reduce silos         → Share ownership with dev teams          │
│  Accept failure       → Error budgets, blameless postmortems    │
│  Implement gradual    → Canary releases, progressive rollout    │
│  Leverage tooling     → Automate toil, self-healing systems     │
│  Measure everything   → SLIs, SLOs, monitoring                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. SLIs, SLOs, and SLAs

### 2.1 Definitions

```
SLI (Service Level Indicator):
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  A quantitative measure of some aspect of the service           │
│                                                                 │
│  Common SLIs:                                                   │
│  ├── Availability: % of successful requests                     │
│  ├── Latency: % of requests faster than threshold               │
│  ├── Throughput: Requests per second                            │
│  ├── Error Rate: % of failed requests                           │
│  └── Durability: % of data preserved over time                  │
│                                                                 │
│  Formula:                                                       │
│  SLI = (Good events / Total events) × 100                       │
│                                                                 │
│  Example:                                                       │
│  Availability SLI = (Successful requests / Total requests)×100  │
│  Latency SLI = (Requests < 200ms / Total requests) × 100        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

SLO (Service Level Objective):
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  A target value for an SLI over a time period                   │
│                                                                 │
│  Examples:                                                      │
│  ├── "99.9% of requests successful over 30 days"                │
│  ├── "95% of requests complete in <200ms over 7 days"           │
│  ├── "99.99% of data remains accessible over 1 year"            │
│  └── "Error rate <0.1% over 24 hours"                           │
│                                                                 │
│  Good SLOs:                                                     │
│  ├── Achievable (not 100%)                                      │
│  ├── Measurable (backed by SLI)                                 │
│  ├── Relevant (matter to users)                                 │
│  └── Time-bound (specific period)                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

SLA (Service Level Agreement):
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  A contract with consequences for missing the target            │
│                                                                 │
│  SLA = SLO + Consequences                                       │
│                                                                 │
│  Example:                                                       │
│  "99.9% availability. If violated, customer receives 10% credit"│
│                                                                 │
│  Relationship:                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                                                          │    │
│  │   SLI (What we measure)                                  │    │
│  │      ↓                                                   │    │
│  │   SLO (Internal target, tighter than SLA)                │    │
│  │      ↓                                                   │    │
│  │   SLA (External promise, with consequences)              │    │
│  │                                                          │    │
│  │   Example:                                               │    │
│  │   SLI: Availability = 99.95%                             │    │
│  │   SLO: 99.9% availability (internal target)              │    │
│  │   SLA: 99.5% availability (customer promise)             │    │
│  │                                                          │    │
│  │   Buffer between SLO and SLA protects against penalties  │    │
│  │                                                          │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Choosing SLIs

```
SLI Selection by Service Type:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  USER-FACING WEB SERVICES:                                      │
│  ├── Availability: % of successful HTTP requests                │
│  ├── Latency: % of requests < threshold (p50, p99)              │
│  └── Throughput: Requests per second                            │
│                                                                 │
│  STORAGE SYSTEMS:                                               │
│  ├── Durability: % of data preserved                            │
│  ├── Availability: % of successful read/write operations        │
│  └── Latency: Time to first byte                                │
│                                                                 │
│  DATA PROCESSING PIPELINES:                                     │
│  ├── Freshness: % of data processed within threshold            │
│  ├── Correctness: % of records processed correctly              │
│  └── Coverage: % of expected data processed                     │
│                                                                 │
│  API SERVICES:                                                  │
│  ├── Availability: % of successful API calls                    │
│  ├── Latency: Response time percentiles                         │
│  └── Correctness: % of responses with valid data                │
│                                                                 │
│  Best Practices:                                                │
│  ├── Start with 2-4 SLIs per service                            │
│  ├── Focus on user-facing symptoms                              │
│  ├── Avoid too many SLIs (cognitive overload)                   │
│  └── Measure at the edge (user's perspective)                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2.3 Implementing SLOs

```yaml
# Example SLO Definition (YAML)

service: payment-api
team: payments
slos:
  - name: availability
    description: "Payment API should be highly available"
    sli:
      type: availability
      metric: |
        sum(rate(http_requests_total{status!~"5.."}[5m])) /
        sum(rate(http_requests_total[5m]))
    target: 99.9
    window: 30d
    burn_rate_alerts:
      - severity: critical
        burn_rate: 14.4  # Exhausts budget in 2 hours
        window: 1h
      - severity: warning
        burn_rate: 6     # Exhausts budget in 5 days
        window: 6h

  - name: latency
    description: "Payment API should respond quickly"
    sli:
      type: latency
      metric: |
        histogram_quantile(0.99,
          sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
        )
    target_threshold: 0.5  # 500ms
    target_percentage: 99  # 99% of requests
    window: 7d

  - name: error_rate
    description: "Payment processing errors should be minimal"
    sli:
      type: error_rate
      metric: |
        sum(rate(payment_errors_total[5m])) /
        sum(rate(payments_total[5m]))
    target: 0.1  # 0.1% error rate
    window: 24h
```

### 2.4 SLO Monitoring with Prometheus

```yaml
# Prometheus Recording Rules for SLOs

groups:
  - name: slo-recording-rules
    interval: 30s
    rules:
      # Total requests
      - record: sli:http_requests:total_rate5m
        expr: sum(rate(http_requests_total[5m]))

      # Successful requests (non-5xx)
      - record: sli:http_requests:success_rate5m
        expr: sum(rate(http_requests_total{status!~"5.."}[5m]))

      # Availability SLI (0-1)
      - record: sli:availability
        expr: sli:http_requests:success_rate5m / sli:http_requests:total_rate5m

      # Latency SLI - requests under 500ms
      - record: sli:latency:under500ms_rate5m
        expr: sum(rate(http_request_duration_seconds_bucket{le="0.5"}[5m]))

      # Latency SLI ratio
      - record: sli:latency_sli
        expr: sli:latency:under500ms_rate5m / sli:http_requests:total_rate5m

  - name: slo-alerting-rules
    rules:
      # Multi-window, multi-burn-rate alerting
      
      # Critical: 2% budget consumption in 1 hour
      - alert: SLOBurnRateCritical
        expr: |
          (
            (1 - sli:availability) > (14.4 * 0.001)  # 14.4x burn rate
            and
            (1 - sli:availability) > (14.4 * 0.001)  # Confirm over short window
          )
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "High SLO burn rate - will exhaust budget in 2 hours"
          runbook_url: "https://runbooks.example.com/slo-burn-rate"

      # Warning: 10% budget consumption in 6 hours
      - alert: SLOBurnRateWarning
        expr: |
          (1 - sli:availability) > (6 * 0.001)  # 6x burn rate
        for: 15m
        labels:
          severity: warning
        annotations:
          summary: "Elevated SLO burn rate - will exhaust budget in 5 days"
```

---

## 3. Error Budgets

### 3.1 Understanding Error Budgets

```
Error Budget Concept:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  If SLO = 99.9% availability over 30 days                       │
│                                                                 │
│  Error Budget = 100% - 99.9% = 0.1%                             │
│                                                                 │
│  In time:                                                       │
│  30 days = 43,200 minutes                                       │
│  Error Budget = 43,200 × 0.001 = 43.2 minutes downtime allowed  │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                                                            │  │
│  │  Month Start                                  Month End    │  │
│  │  ├──────────────────────────────────────────────────────┤  │  │
│  │  │▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓░░░░░░░░░░░░░░│  │  │
│  │  │◄─────── Consumed (30 min) ──────────►│◄─ Remaining ─►│  │  │
│  │                                          (13.2 min)       │  │
│  │                                                            │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  Error Budget = Reliability investment currency                 │
│  ├── Spent by: Incidents, bugs, deployments, experiments       │
│  └── When exhausted: Freeze features, focus on reliability     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 Error Budget Policies

```
Error Budget Policy Example:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  BUDGET STATUS         ACTIONS                                  │
│  ─────────────────────────────────────────────────────────────  │
│  > 50% remaining       Normal development velocity              │
│                        Standard change process                  │
│                                                                 │
│  25-50% remaining      Increased testing required               │
│                        Canary deployments mandatory             │
│                        Review recent incidents                  │
│                                                                 │
│  10-25% remaining      Feature freeze consideration             │
│                        Only P1 bugs and reliability work        │
│                        Require SRE approval for changes         │
│                                                                 │
│  < 10% remaining       Feature freeze enacted                   │
│                        All hands on reliability                 │
│                        Postmortem required for any incident     │
│                        Daily budget review meetings             │
│                                                                 │
│  Budget exhausted      Complete change freeze                   │
│                        Only emergency fixes allowed             │
│                        Mandatory reliability improvements       │
│                        Exec-level escalation                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

Error Budget Calculation:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  # Daily error budget tracking                                  │
│                                                                 │
│  SLO = 99.9%                                                    │
│  Window = 30 days                                               │
│  Budget = 0.1% = 43.2 minutes                                   │
│                                                                 │
│  Day 1:  2 min downtime  → Budget: 41.2 min (95.4% remaining)   │
│  Day 5:  5 min downtime  → Budget: 36.2 min (83.8% remaining)   │
│  Day 10: 0 min downtime  → Budget: 36.2 min (83.8% remaining)   │
│  Day 15: 20 min incident → Budget: 16.2 min (37.5% remaining)   │
│  Day 20: 10 min incident → Budget: 6.2 min  (14.4% remaining)   │
│                             ↑                                   │
│                             Feature freeze triggered!           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 3.3 Error Budget Dashboard

```yaml
# Grafana Dashboard JSON snippet for Error Budget

{
  "panels": [
    {
      "title": "Error Budget Remaining",
      "type": "gauge",
      "targets": [
        {
          "expr": "(1 - ((1 - avg_over_time(sli:availability[30d])) / 0.001)) * 100",
          "legendFormat": "Budget Remaining %"
        }
      ],
      "thresholds": {
        "mode": "absolute",
        "steps": [
          { "color": "red", "value": 0 },
          { "color": "orange", "value": 25 },
          { "color": "yellow", "value": 50 },
          { "color": "green", "value": 75 }
        ]
      }
    },
    {
      "title": "Error Budget Burn Rate",
      "type": "timeseries",
      "targets": [
        {
          "expr": "(1 - sli:availability) / 0.001",
          "legendFormat": "Current Burn Rate (1 = normal)"
        }
      ],
      "thresholds": [
        { "value": 1, "colorMode": "ok" },
        { "value": 6, "colorMode": "warning" },
        { "value": 14, "colorMode": "critical" }
      ]
    },
    {
      "title": "Time Until Budget Exhaustion",
      "type": "stat",
      "targets": [
        {
          "expr": "((0.001 - (1 - avg_over_time(sli:availability[30d]))) * 30 * 24 * 60) / (1 - sli:availability)",
          "legendFormat": "Minutes until exhausted at current burn rate"
        }
      ]
    }
  ]
}
```

---

## 4. Toil Reduction

### 4.1 Identifying Toil

```
What is Toil?
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Toil is the kind of work tied to running a production service  │
│  that tends to be manual, repetitive, automatable, tactical,    │
│  devoid of enduring value, and that scales linearly as service  │
│  grows.                                                         │
│                                                                 │
│  Characteristics of Toil:                                       │
│  ├── Manual: Human performs the action                          │
│  ├── Repetitive: Done over and over                             │
│  ├── Automatable: Could be done by a machine                    │
│  ├── Tactical: Reactive, not strategic                          │
│  ├── No enduring value: Doesn't improve the service             │
│  └── Scales with service: O(n) with growth                      │
│                                                                 │
│  Examples of Toil:                                              │
│  ├── Manually running deployment scripts                        │
│  ├── Restarting services after failures                         │
│  ├── Adding users/permissions manually                          │
│  ├── Copying files between servers                              │
│  ├── Manually scaling resources                                 │
│  ├── Responding to false-positive alerts                        │
│  └── Manual log analysis                                        │
│                                                                 │
│  NOT Toil (legitimate work):                                    │
│  ├── Strategic architecture decisions                           │
│  ├── Writing automation code                                    │
│  ├── Designing monitoring systems                               │
│  ├── Incident response (initial)                                │
│  └── Code reviews                                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 Toil Measurement

```
Toil Tracking:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Google's Target: SREs spend ≤50% time on toil                 │
│                                                                 │
│  Tracking Method:                                               │
│  1. Log activities for 2 weeks                                  │
│  2. Categorize each activity                                    │
│  3. Calculate toil percentage                                   │
│                                                                 │
│  Activity Categories:                                           │
│  ├── Toil (manual, repetitive, automatable)                     │
│  ├── Engineering (automation, architecture, tooling)            │
│  ├── Overhead (meetings, email, admin)                          │
│  └── On-call (incident response)                                │
│                                                                 │
│  Example Tracking Sheet:                                        │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ Task                    │ Time │ Category  │ Automatable? │  │
│  │─────────────────────────────────────────────────────────────│  │
│  │ Deploy to production    │ 2h   │ Toil      │ Yes          │  │
│  │ Investigate alert       │ 1h   │ On-call   │ Partial      │  │
│  │ Add new user accounts   │ 30m  │ Toil      │ Yes          │  │
│  │ Write deployment script │ 4h   │ Engineer  │ N/A          │  │
│  │ Team meeting            │ 1h   │ Overhead  │ No           │  │
│  │ Restart crashed service │ 15m  │ Toil      │ Yes          │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  Weekly Summary:                                                │
│  Toil: 15 hours (37.5%)  ← Good, under 50%                     │
│  Engineering: 16 hours (40%)                                    │
│  Overhead: 5 hours (12.5%)                                      │
│  On-call: 4 hours (10%)                                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.3 Automating Toil

```
Toil Elimination Examples:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  BEFORE: Manual Deployment                                      │
│  ├── SSH to server                                              │
│  ├── Pull latest code                                           │
│  ├── Run database migrations                                    │
│  ├── Restart application                                        │
│  ├── Verify health                                              │
│  └── Time: 30 min, Error-prone                                  │
│                                                                 │
│  AFTER: Automated Pipeline                                      │
│  ├── git push triggers Jenkins                                  │
│  ├── Automated tests run                                        │
│  ├── ArgoCD syncs to cluster                                    │
│  ├── Progressive rollout                                        │
│  ├── Automatic rollback on failure                              │
│  └── Time: 5 min hands-off                                      │
│                                                                 │
│  ───────────────────────────────────────────────────────────────│
│                                                                 │
│  BEFORE: Manual Certificate Renewal                             │
│  ├── Calendar reminder                                          │
│  ├── Generate CSR                                               │
│  ├── Submit to CA                                               │
│  ├── Download certificate                                       │
│  ├── Install on servers                                         │
│  └── Time: 1-2 hours, risk of expiry                            │
│                                                                 │
│  AFTER: cert-manager in Kubernetes                              │
│  ├── cert-manager watches expiry                                │
│  ├── Automatically requests renewal                             │
│  ├── Installs new certificate                                   │
│  └── Time: 0 min, always valid                                  │
│                                                                 │
│  ───────────────────────────────────────────────────────────────│
│                                                                 │
│  BEFORE: Manual Capacity Scaling                                │
│  ├── Monitor dashboards                                         │
│  ├── Predict traffic spikes                                     │
│  ├── Manually add nodes                                         │
│  └── Time: Hours of attention                                   │
│                                                                 │
│  AFTER: Kubernetes HPA + Cluster Autoscaler                     │
│  ├── HPA scales pods based on metrics                           │
│  ├── Cluster Autoscaler adds nodes                              │
│  ├── Automatic scale-down when quiet                            │
│  └── Time: 0 min, reactive to actual load                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Incident Management

### 5.1 Incident Lifecycle

```
Incident Management Process:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. DETECTION                                                   │
│     ├── Monitoring alerts                                       │
│     ├── Customer reports                                        │
│     └── Internal observation                                    │
│                                                                 │
│  2. TRIAGE                                                      │
│     ├── Assess severity (SEV1-4)                                │
│     ├── Identify affected services                              │
│     └── Initial impact assessment                               │
│                                                                 │
│  3. RESPONSE                                                    │
│     ├── Assign Incident Commander                               │
│     ├── Establish communication channel                         │
│     ├── Begin investigation                                     │
│     └── Implement mitigation                                    │
│                                                                 │
│  4. RESOLUTION                                                  │
│     ├── Apply fix                                               │
│     ├── Verify recovery                                         │
│     └── Monitor for recurrence                                  │
│                                                                 │
│  5. CLOSURE                                                     │
│     ├── Confirm service restored                                │
│     ├── Update status page                                      │
│     ├── Notify stakeholders                                     │
│     └── Schedule postmortem                                     │
│                                                                 │
│  6. POSTMORTEM                                                  │
│     ├── Document timeline                                       │
│     ├── Root cause analysis                                     │
│     ├── Identify action items                                   │
│     └── Share learnings                                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 Severity Levels

```
Incident Severity Definitions:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  SEV1 - Critical                                                │
│  ├── Impact: Complete service outage, data loss risk            │
│  ├── Users: All users affected                                  │
│  ├── Response: Immediate, all-hands                             │
│  ├── Communication: Every 15 minutes                            │
│  ├── Escalation: Exec notification within 15 min                │
│  └── Example: Production database down                          │
│                                                                 │
│  SEV2 - High                                                    │
│  ├── Impact: Major feature unavailable, significant degradation │
│  ├── Users: Large subset affected                               │
│  ├── Response: Within 30 minutes                                │
│  ├── Communication: Every 30 minutes                            │
│  ├── Escalation: Manager notification                           │
│  └── Example: Payment processing failing                        │
│                                                                 │
│  SEV3 - Medium                                                  │
│  ├── Impact: Minor feature unavailable, workaround exists       │
│  ├── Users: Small subset affected                               │
│  ├── Response: Within 4 hours                                   │
│  ├── Communication: Every 2 hours                               │
│  └── Example: Report generation slow                            │
│                                                                 │
│  SEV4 - Low                                                     │
│  ├── Impact: Cosmetic issue, no functional impact               │
│  ├── Users: Minimal or none                                     │
│  ├── Response: Next business day                                │
│  ├── Communication: As needed                                   │
│  └── Example: Typo in error message                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.3 Incident Roles

```
Incident Response Roles:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  INCIDENT COMMANDER (IC)                                        │
│  ├── Coordinates response                                       │
│  ├── Makes decisions                                            │
│  ├── Delegates tasks                                            │
│  ├── Owns communication                                         │
│  └── Can be handed off                                          │
│                                                                 │
│  COMMUNICATIONS LEAD                                            │
│  ├── Updates status page                                        │
│  ├── Notifies stakeholders                                      │
│  ├── Handles customer communication                             │
│  └── Documents timeline                                         │
│                                                                 │
│  OPERATIONS LEAD                                                │
│  ├── Performs technical investigation                           │
│  ├── Implements mitigations                                     │
│  ├── Applies fixes                                              │
│  └── Coordinates with other teams                               │
│                                                                 │
│  SUBJECT MATTER EXPERTS (SMEs)                                  │
│  ├── Provide domain expertise                                   │
│  ├── Advise on fixes                                            │
│  └── May take over Ops Lead role                                │
│                                                                 │
│  SCRIBE                                                         │
│  ├── Documents everything                                       │
│  ├── Records timeline                                           │
│  ├── Notes decisions                                            │
│  └── Prepares postmortem material                               │
│                                                                 │
│  Example Incident Channel:                                      │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ #incident-2024-01-15-payment-outage                     │    │
│  │                                                          │    │
│  │ IC: @sarah                                               │    │
│  │ Comms: @mike                                             │    │
│  │ Ops: @alex                                               │    │
│  │ Scribe: @lisa                                            │    │
│  │                                                          │    │
│  │ Status: INVESTIGATING                                    │    │
│  │ Impact: Payments failing for 20% of users                │    │
│  │ Start Time: 14:30 UTC                                    │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. On-Call Best Practices

### 6.1 On-Call Structure

```
On-Call Rotation:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Rotation Models:                                               │
│                                                                 │
│  1. WEEKLY ROTATION                                             │
│     ├── Engineer A: Week 1                                      │
│     ├── Engineer B: Week 2                                      │
│     ├── Engineer C: Week 3                                      │
│     └── Repeat                                                  │
│                                                                 │
│  2. PRIMARY + SECONDARY                                         │
│     ├── Primary: First responder                                │
│     ├── Secondary: Backup, escalation                           │
│     └── Primary becomes Secondary next rotation                 │
│                                                                 │
│  3. FOLLOW-THE-SUN                                              │
│     ├── US team: 9am-5pm PST                                    │
│     ├── EU team: 9am-5pm CET                                    │
│     ├── APAC team: 9am-5pm JST                                  │
│     └── No one on-call during night                             │
│                                                                 │
│  4. TIERED ON-CALL                                              │
│     ├── L1: Operations team (first response)                    │
│     ├── L2: Service owners (escalation)                         │
│     └── L3: Subject matter experts (complex issues)             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 6.2 On-Call Health

```
Healthy On-Call Metrics:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  ALERT VOLUME                                                   │
│  ├── Target: <2 pages per on-call shift                         │
│  ├── Measure: Pages per week, per service                       │
│  └── Action: Tune alerts if too high                            │
│                                                                 │
│  FALSE POSITIVE RATE                                            │
│  ├── Target: <5% of alerts are false positives                  │
│  ├── Measure: Alerts that required no action                    │
│  └── Action: Fix noisy alerts or remove them                    │
│                                                                 │
│  TIME TO ACKNOWLEDGE                                            │
│  ├── Target: <5 minutes for SEV1                                │
│  ├── Measure: Time from page to ack                             │
│  └── Action: Escalate if not acknowledged                       │
│                                                                 │
│  TIME TO RESOLVE                                                │
│  ├── Target: Varies by severity                                 │
│  ├── Measure: Time from page to resolution                      │
│  └── Action: Improve runbooks, automation                       │
│                                                                 │
│  ON-CALL BURDEN                                                 │
│  ├── Target: ≤25% time on interrupt work                       │
│  ├── Measure: Hours spent on pages                              │
│  └── Action: Distribute load, automate                          │
│                                                                 │
│  Burnout Prevention:                                            │
│  ├── Max 1 week on-call per month                               │
│  ├── Compensatory time off after heavy shifts                   │
│  ├── No back-to-back rotations                                  │
│  ├── Shadow period for new on-callers                           │
│  └── Regular review of alert fatigue                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 6.3 Runbooks

```markdown
# Runbook Template: Service Unavailable

## Overview
This runbook addresses alerts for the payment-service being unavailable.

## Alert
- **Name**: PaymentServiceDown
- **Severity**: SEV1
- **Description**: Payment service health check failing

## Impact
- Users cannot complete purchases
- Revenue impact: ~$10K/minute

## Quick Actions (Try First)
1. Check if pods are running:
   ```bash
   kubectl get pods -n payments -l app=payment-service
   ```

2. Check recent deployments:
   ```bash
   kubectl rollout history deployment/payment-service -n payments
   ```

3. Rollback if recent deployment:
   ```bash
   kubectl rollout undo deployment/payment-service -n payments
   ```

## Investigation Steps

### Check Service Health
```bash
# Pod status
kubectl describe pods -n payments -l app=payment-service

# Recent logs
kubectl logs -n payments -l app=payment-service --tail=100

# Events
kubectl get events -n payments --sort-by='.lastTimestamp'
```

### Check Dependencies
1. Database connectivity:
   ```bash
   kubectl exec -it deployment/payment-service -n payments -- \
     nc -zv postgres.database 5432
   ```

2. Redis connectivity:
   ```bash
   kubectl exec -it deployment/payment-service -n payments -- \
     nc -zv redis.cache 6379
   ```

### Common Causes
| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| OOMKilled | Memory leak | Increase memory, investigate leak |
| CrashLoopBackOff | Config error | Check configmaps, secrets |
| Pending | Resource shortage | Scale cluster or reduce requests |
| ImagePullBackOff | Registry issue | Check ACR, image exists |

## Escalation
- If not resolved in 15 minutes: Page secondary
- If database issue: Page DBA team
- If network issue: Page network team

## Resolution
1. Verify pods healthy: `kubectl get pods -n payments`
2. Verify endpoint responding: `curl https://api.example.com/payments/health`
3. Resolve incident in PagerDuty

## Post-Incident
- [ ] Update incident channel
- [ ] Schedule postmortem if SEV1/2
- [ ] Create tickets for any follow-up work
```

---

## 7. Postmortems

### 7.1 Blameless Postmortems

```
Blameless Postmortem Principles:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  CORE PRINCIPLE:                                                │
│  Assume everyone did their best given what they knew at the     │
│  time, the information they had, and the circumstances.         │
│                                                                 │
│  GOALS:                                                         │
│  ├── Learn from incidents                                       │
│  ├── Prevent recurrence                                         │
│  ├── Improve systems and processes                              │
│  └── Share knowledge across organization                        │
│                                                                 │
│  WHAT TO AVOID:                                                 │
│  ├── "Bob made a mistake"                                       │
│  ├── "Human error was the root cause"                           │
│  ├── "We need more training"                                    │
│  └── Punishing individuals for mistakes                         │
│                                                                 │
│  WHAT TO DO:                                                    │
│  ├── "The system allowed this action without safeguards"        │
│  ├── "The deployment process didn't catch this"                 │
│  ├── "The runbook was unclear about this scenario"              │
│  └── Focus on systemic improvements                             │
│                                                                 │
│  Example Reframing:                                             │
│  Before: "Engineer deployed bad code to production"             │
│  After: "The CI/CD pipeline didn't catch the regression,        │
│          and the deployment lacked a canary phase"              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 7.2 Postmortem Template

```markdown
# Postmortem: Payment Service Outage

**Date**: 2024-01-15
**Duration**: 45 minutes (14:30 - 15:15 UTC)
**Severity**: SEV1
**Author**: Sarah Chen
**Status**: Complete

## Summary
Payment service was unavailable for 45 minutes due to a database 
connection pool exhaustion caused by a slow query introduced in 
a recent deployment.

## Impact
- 45 minutes of payment processing unavailable
- ~2,000 failed transactions
- Estimated revenue impact: $150,000
- Customer support received ~500 tickets

## Timeline (All times UTC)
| Time | Event |
|------|-------|
| 14:00 | Deployment v2.15.0 completed |
| 14:30 | Alerts fire: Payment error rate >5% |
| 14:32 | On-call engineer acknowledges |
| 14:35 | Incident declared, war room started |
| 14:40 | Identified connection pool exhaustion |
| 14:45 | Identified slow query in new code |
| 14:50 | Decision to rollback |
| 14:55 | Rollback initiated |
| 15:10 | Rollback complete, monitoring |
| 15:15 | Service restored, incident resolved |

## Root Cause
The v2.15.0 deployment included a new reporting query that lacked 
proper indexing. Under production load, this query took 30+ seconds 
to execute, holding database connections. The connection pool (size: 20)
was exhausted within 30 minutes of deployment, causing new requests 
to fail.

## Contributing Factors
1. Query was not tested with production data volumes
2. No database query performance testing in CI/CD
3. Connection pool size not monitored
4. No canary deployment for this service

## What Went Well
- Alert fired within seconds of threshold breach
- On-call response was immediate (<5 min)
- Root cause identified quickly (15 min)
- Rollback process was smooth

## What Went Wrong
- Slow query wasn't caught in review
- No connection pool monitoring
- Took 20 min to decide to rollback (should be faster)
- Customers weren't notified for 20 minutes

## Action Items
| Action | Owner | Priority | Due Date |
|--------|-------|----------|----------|
| Add query performance tests to CI | Alex | P1 | 2024-01-22 |
| Add connection pool metrics to dashboard | Sarah | P1 | 2024-01-20 |
| Implement canary deployments for payments | DevOps | P2 | 2024-02-01 |
| Add index to reporting table | Mike | P1 | 2024-01-17 |
| Update rollback decision criteria | SRE Team | P2 | 2024-01-25 |
| Improve customer notification process | Support | P2 | 2024-01-30 |

## Lessons Learned
1. Queries should be tested with production-like data volumes
2. Connection pool exhaustion is a leading indicator of issues
3. Faster rollback decisions reduce impact duration
4. Canary deployments would have limited blast radius

## Supporting Documents
- [Incident Slack thread](link)
- [Deployment logs](link)
- [Grafana dashboard during incident](link)
```

---

## 8. Reliability Patterns

### 8.1 Graceful Degradation

```
Graceful Degradation Patterns:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. CIRCUIT BREAKER                                             │
│     When dependency fails, stop calling it                      │
│     ├── Prevents cascade failures                               │
│     ├── Allows dependency to recover                            │
│     └── Returns fallback or error quickly                       │
│                                                                 │
│  2. BULKHEAD                                                    │
│     Isolate failures to prevent spread                          │
│     ├── Separate thread pools per dependency                    │
│     ├── Separate pods per criticality                           │
│     └── Rate limit per tenant                                   │
│                                                                 │
│  3. FALLBACK                                                    │
│     Provide degraded experience when optimal fails              │
│     ├── Cache: Return stale data if fresh unavailable           │
│     ├── Default: Return sensible default values                 │
│     └── Queue: Accept request for later processing              │
│                                                                 │
│  4. FEATURE FLAGS                                               │
│     Disable features under load                                 │
│     ├── Disable non-critical features                           │
│     ├── Simplify complex operations                             │
│     └── Reduce database load                                    │
│                                                                 │
│  Example Implementation:                                        │
│                                                                 │
│  def get_recommendations(user_id):                              │
│      try:                                                       │
│          # Primary: ML-based recommendations                    │
│          return recommendation_service.get(user_id, timeout=2s) │
│      except (Timeout, CircuitOpen):                             │
│          # Fallback 1: Cached recommendations                   │
│          cached = cache.get(f"recs:{user_id}")                  │
│          if cached:                                             │
│              return cached                                      │
│          # Fallback 2: Popular items                            │
│          return get_popular_items()                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 8.2 Load Shedding

```
Load Shedding Strategies:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  When: System is overloaded, can't handle all requests          │
│  Goal: Serve some requests well rather than all requests poorly │
│                                                                 │
│  Strategies:                                                    │
│                                                                 │
│  1. PRIORITY-BASED                                              │
│     Serve high-priority requests, reject low-priority           │
│     ├── Paid users over free users                              │
│     ├── Health checks over analytics                            │
│     └── Writes over reads (or vice versa)                       │
│                                                                 │
│  2. FAIR QUEUING                                                │
│     Equal share per client/tenant                               │
│     └── Prevents one client from monopolizing                   │
│                                                                 │
│  3. ADAPTIVE RATE LIMITING                                      │
│     Reduce limits as load increases                             │
│     └── Gradual degradation                                     │
│                                                                 │
│  4. REQUEST COALESCING                                          │
│     Combine duplicate requests                                  │
│     └── One DB query for 100 identical requests                 │
│                                                                 │
│  Implementation:                                                │
│                                                                 │
│  @app.middleware                                                │
│  async def load_shedding(request, call_next):                   │
│      # Check current load                                       │
│      if system_load > 0.9:  # 90% capacity                      │
│          if request.priority == "low":                          │
│              return Response(status=503,                        │
│                  headers={"Retry-After": "30"})                 │
│                                                                 │
│      if queue_depth > 1000:                                     │
│          return Response(status=503,                            │
│              body="Server overloaded, please retry")            │
│                                                                 │
│      return await call_next(request)                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 9. Change Management

### 9.1 Progressive Rollouts

```
Progressive Rollout Strategies:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. CANARY DEPLOYMENT                                           │
│     ├── Deploy to small subset (1-5%)                           │
│     ├── Monitor for errors                                      │
│     ├── Gradually increase (10%, 25%, 50%, 100%)                │
│     └── Automatic rollback on error spike                       │
│                                                                 │
│     ┌───────────────────────────────────────────────────────┐   │
│     │                                                        │   │
│     │  Traffic ──► Load Balancer                             │   │
│     │                   │                                    │   │
│     │         ┌─────────┴─────────┐                          │   │
│     │         │ 95%               │ 5%                       │   │
│     │         ▼                   ▼                          │   │
│     │    [v1.0] [v1.0] [v1.0]   [v1.1]  ← Canary             │   │
│     │                                                        │   │
│     └───────────────────────────────────────────────────────┘   │
│                                                                 │
│  2. BLUE-GREEN DEPLOYMENT                                       │
│     ├── Two identical environments                              │
│     ├── Deploy to inactive (Green)                              │
│     ├── Switch traffic all at once                              │
│     └── Keep Blue for instant rollback                          │
│                                                                 │
│  3. FEATURE FLAGS                                               │
│     ├── Deploy code but keep feature off                        │
│     ├── Enable for percentage of users                          │
│     ├── Target specific user segments                           │
│     └── Kill switch for instant disable                         │
│                                                                 │
│  4. RING DEPLOYMENT                                             │
│     ├── Ring 0: Internal users                                  │
│     ├── Ring 1: Beta users / early adopters                     │
│     ├── Ring 2: 10% of production                               │
│     ├── Ring 3: 50% of production                               │
│     └── Ring 4: 100% of production                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 9.2 Change Freeze Policies

```
Change Management Calendar:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  REGULAR PERIODS:                                               │
│  ├── Monday-Thursday: Normal change window                      │
│  ├── Friday after 2pm: No changes (weekend risk)                │
│  ├── Weekend: Emergency only                                    │
│  └── On-call transition: No changes during handoff              │
│                                                                 │
│  BLACKOUT PERIODS:                                              │
│  ├── Major retail events (Black Friday, Cyber Monday)           │
│  ├── End of quarter / fiscal year close                         │
│  ├── Major sports events (for relevant businesses)              │
│  └── Company all-hands / critical meetings                      │
│                                                                 │
│  EXCEPTION PROCESS:                                             │
│  1. Emergency change request                                    │
│  2. Approval from on-call lead + manager                        │
│  3. Extra monitoring during deployment                          │
│  4. Immediate rollback plan ready                               │
│                                                                 │
│  Calendar Example:                                              │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ Mon   Tue   Wed   Thu   Fri   Sat   Sun                  │  │
│  │  ✓     ✓     ✓     ✓    ⚠️    ❌    ❌                    │  │
│  │                                                           │  │
│  │  ✓ = Normal changes allowed                               │  │
│  │  ⚠️ = Before 2pm only                                     │  │
│  │  ❌ = Emergency only                                       │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 10. SRE Tools and Automation

### 10.1 Self-Healing Systems

```yaml
# Kubernetes Self-Healing Configuration

apiVersion: apps/v1
kind: Deployment
metadata:
  name: resilient-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: resilient-app
  template:
    metadata:
      labels:
        app: resilient-app
    spec:
      containers:
      - name: app
        image: myapp:v1.0
        
        # Liveness: Restart if unhealthy
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          failureThreshold: 3
        
        # Readiness: Remove from LB if not ready
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
          failureThreshold: 3
        
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
      
      # Restart policy
      restartPolicy: Always
      
      # Spread across nodes/zones
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: ScheduleAnyway
        labelSelector:
          matchLabels:
            app: resilient-app

---
# Pod Disruption Budget
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: resilient-app-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: resilient-app

---
# Horizontal Pod Autoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: resilient-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: resilient-app
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Pods
        value: 1
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Pods
        value: 2
        periodSeconds: 60
```

### 10.2 Automated Remediation

```python
# Example: Automated Pod Restart on High Error Rate

import kubernetes
from prometheus_api_client import PrometheusConnect

def check_and_remediate():
    # Connect to Prometheus
    prom = PrometheusConnect(url="http://prometheus:9090")
    
    # Query error rate
    query = '''
        sum(rate(http_requests_total{status=~"5.."}[5m])) /
        sum(rate(http_requests_total[5m]))
    '''
    result = prom.custom_query(query)
    error_rate = float(result[0]['value'][1])
    
    if error_rate > 0.1:  # More than 10% errors
        print(f"High error rate detected: {error_rate:.2%}")
        
        # Load Kubernetes config
        kubernetes.config.load_incluster_config()
        v1 = kubernetes.client.CoreV1Api()
        
        # Get pods with errors (oldest first)
        pods = v1.list_namespaced_pod(
            namespace="production",
            label_selector="app=myapp"
        )
        
        # Delete oldest pod (Deployment will recreate)
        oldest_pod = min(pods.items, 
                         key=lambda p: p.metadata.creation_timestamp)
        
        print(f"Restarting pod: {oldest_pod.metadata.name}")
        v1.delete_namespaced_pod(
            name=oldest_pod.metadata.name,
            namespace="production"
        )
        
        # Alert team
        send_slack_notification(
            channel="#sre-alerts",
            message=f"Auto-remediation: Restarted {oldest_pod.metadata.name} "
                    f"due to {error_rate:.2%} error rate"
        )

# Run every 5 minutes
if __name__ == "__main__":
    check_and_remediate()
```

---

## Summary

SRE is a discipline that brings software engineering practices to operations:

1. **SLIs/SLOs/SLAs**: Quantify reliability targets and measure against them
2. **Error Budgets**: Balance reliability with development velocity
3. **Toil Reduction**: Automate repetitive work, target ≤50% toil
4. **Incident Management**: Structured response with clear roles
5. **On-Call**: Sustainable rotations with healthy metrics
6. **Postmortems**: Blameless learning from failures
7. **Reliability Patterns**: Circuit breakers, fallbacks, load shedding
8. **Change Management**: Progressive rollouts, change freezes
9. **Automation**: Self-healing systems, automated remediation

The ultimate goal is to run reliable systems while maintaining development velocity, using error budgets as the currency to balance these competing needs.

---

**Next Part**: [Part 11: GitOps with ArgoCD and Flux](./DevOps-Complete-Reference-Guide-Part11-GitOps.md)
