# Azure Monitoring & Management - Complete Guide
## Monitor, Log Analytics, Application Insights, Alerts, Automation

---

# 1. AZURE MONITOR

## What It Is
Comprehensive monitoring solution for collecting, analyzing, and acting on telemetry from cloud and on-premises environments.

## Data Types
```
DATA COLLECTED:
┌─────────────────────────────────────────────────────────────────┐
│ METRICS:            Numerical values at regular intervals      │
│                     CPU, memory, disk, network                  │
│                     Near real-time (1 min granularity)          │
│                                                                 │
│ LOGS:               Detailed records with timestamps           │
│                     Events, traces, performance data           │
│                     Stored in Log Analytics                    │
│                                                                 │
│ TRACES:             Distributed tracing data                   │
│                     Request flow across services               │
│                     Application Insights                       │
└─────────────────────────────────────────────────────────────────┘
```

## Architecture
```
AZURE MONITOR ARCHITECTURE:
┌─────────────────────────────────────────────────────────────────┐
│                         DATA SOURCES                            │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │   VMs   │ │   AKS   │ │ AppSvc  │ │   SQL   │ │ Custom  │   │
│  └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘   │
│       │          │          │          │          │            │
│       └──────────┴──────────┼──────────┴──────────┘            │
│                             ▼                                   │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    AZURE MONITOR                          │  │
│  │  ┌─────────────────┐    ┌─────────────────┐              │  │
│  │  │     Metrics     │    │      Logs       │              │  │
│  │  │   (Time-series) │    │ (Log Analytics) │              │  │
│  │  └────────┬────────┘    └────────┬────────┘              │  │
│  └───────────┼──────────────────────┼───────────────────────┘  │
│              │                      │                           │
│              ▼                      ▼                           │
│  ┌─────────────────┐    ┌─────────────────┐    ┌────────────┐  │
│  │     Alerts      │    │    Dashboards   │    │  Workbooks │  │
│  └─────────────────┘    └─────────────────┘    └────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

## CLI Commands

```bash
# Enable diagnostics on resource
az monitor diagnostic-settings create \
  --name myDiagSettings \
  --resource /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Compute/virtualMachines/myVM \
  --workspace /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.OperationalInsights/workspaces/myWorkspace \
  --metrics '[{"category": "AllMetrics", "enabled": true}]' \
  --logs '[{"category": "Administrative", "enabled": true}]'

# Get metrics
az monitor metrics list \
  --resource /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Compute/virtualMachines/myVM \
  --metric "Percentage CPU" \
  --interval PT1H

# List available metrics for resource
az monitor metrics list-definitions \
  --resource /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Compute/virtualMachines/myVM
```

---

# 2. LOG ANALYTICS WORKSPACE

## What It Is
Central repository for log data. Uses Kusto Query Language (KQL) for analysis.

## Key Concepts
```
LOG ANALYTICS:
┌─────────────────────────────────────────────────────────────────┐
│ WORKSPACE:         Central log repository                      │
│ TABLES:            Categorized log data                        │
│ KQL:               Query language                              │
│ RETENTION:         How long logs are kept (30-730 days)        │
│ DATA INGESTION:    Charged per GB ingested                     │
└─────────────────────────────────────────────────────────────────┘
```

## CLI Commands

```bash
# Create workspace
az monitor log-analytics workspace create \
  --resource-group myRG \
  --workspace-name myWorkspace \
  --location eastus \
  --retention-time 90

# Get workspace ID
az monitor log-analytics workspace show \
  --resource-group myRG \
  --workspace-name myWorkspace \
  --query customerId -o tsv

# Query logs
az monitor log-analytics query \
  --workspace <workspace-id> \
  --analytics-query "AzureDiagnostics | where TimeGenerated > ago(1h) | limit 10"
```

## KQL Examples

```kql
// Basic query - get recent events
AzureDiagnostics
| where TimeGenerated > ago(1h)
| limit 100

// Filter and project
AzureDiagnostics
| where ResourceType == "VIRTUALMACHINES"
| where Level == "Error"
| project TimeGenerated, Resource, Category, OperationName, Message

// Aggregate - count by category
AzureDiagnostics
| where TimeGenerated > ago(24h)
| summarize Count = count() by Category
| order by Count desc

// Time chart
AzureDiagnostics
| where TimeGenerated > ago(7d)
| summarize Count = count() by bin(TimeGenerated, 1h), Category
| render timechart

// Container logs (AKS)
ContainerLog
| where TimeGenerated > ago(1h)
| where LogEntry contains "error"
| project TimeGenerated, ContainerID, LogEntry

// Kubernetes events
KubeEvents
| where TimeGenerated > ago(24h)
| where Reason == "FailedScheduling" or Reason == "BackOff"
| project TimeGenerated, Name, Reason, Message

// Performance data
Perf
| where TimeGenerated > ago(1h)
| where CounterName == "% Processor Time"
| summarize AvgCPU = avg(CounterValue) by Computer, bin(TimeGenerated, 5m)
| render timechart

// Join tables
Heartbeat
| where TimeGenerated > ago(1h)
| join kind=inner (
    Perf
    | where TimeGenerated > ago(1h)
    | where CounterName == "% Processor Time"
) on Computer
```

---

# 3. APPLICATION INSIGHTS

## What It Is
Application Performance Management (APM) service for live web applications. Auto-detects anomalies, provides analytics, and helps diagnose issues.

## Key Features
```
CAPABILITIES:
┌─────────────────────────────────────────────────────────────────┐
│ LIVE METRICS:       Real-time performance                      │
│ AVAILABILITY:       Synthetic monitoring                       │
│ FAILURES:           Exception and error analysis               │
│ PERFORMANCE:        Response times, dependencies               │
│ USERS:              Usage analytics                            │
│ APPLICATION MAP:    Visual dependency map                      │
│ DISTRIBUTED TRACING: End-to-end request tracking               │
│ SMART DETECTION:    Auto anomaly detection                     │
└─────────────────────────────────────────────────────────────────┘
```

## CLI Commands

```bash
# Create Application Insights
az monitor app-insights component create \
  --app myAppInsights \
  --location eastus \
  --resource-group myRG \
  --kind web \
  --application-type web

# Get instrumentation key
az monitor app-insights component show \
  --app myAppInsights \
  --resource-group myRG \
  --query instrumentationKey -o tsv

# Get connection string
az monitor app-insights component show \
  --app myAppInsights \
  --resource-group myRG \
  --query connectionString -o tsv

# Query Application Insights
az monitor app-insights query \
  --app myAppInsights \
  --resource-group myRG \
  --analytics-query "requests | where timestamp > ago(1h) | summarize count() by resultCode"
```

## Instrumentation

```javascript
// Node.js
const appInsights = require("applicationinsights");
appInsights.setup("<connection-string>").start();

// Custom event
appInsights.defaultClient.trackEvent({
  name: "OrderPlaced",
  properties: { orderId: "12345" }
});
```

```java
// Java - application.properties
applicationinsights.connection.string=<connection-string>

// Or with agent (recommended)
// Add -javaagent:applicationinsights-agent.jar to JVM args
```

---

# 4. AZURE ALERTS

## What It Is
Notification system that monitors metrics and logs, and triggers actions when conditions are met.

## Alert Types
```
ALERT TYPES:
┌─────────────────────────────────────────────────────────────────┐
│ METRIC ALERTS:      Based on metric thresholds                 │
│                     CPU > 80%, Memory > 90%                    │
│                                                                 │
│ LOG ALERTS:         Based on log query results                 │
│                     Error count > 10 in 5 min                  │
│                                                                 │
│ ACTIVITY LOG ALERTS: Azure resource operations                 │
│                     VM deleted, role assigned                  │
│                                                                 │
│ SMART DETECTION:    ML-based anomaly detection                 │
│                     Automatic in App Insights                  │
└─────────────────────────────────────────────────────────────────┘
```

## CLI Commands

```bash
# Create action group (where to send alerts)
az monitor action-group create \
  --resource-group myRG \
  --name myActionGroup \
  --short-name myAG \
  --email-receiver name="Admin" address="admin@contoso.com" \
  --sms-receiver name="OnCall" country-code="1" phone-number="1234567890"

# Create metric alert
az monitor metrics alert create \
  --name "High CPU Alert" \
  --resource-group myRG \
  --scopes /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Compute/virtualMachines/myVM \
  --condition "avg Percentage CPU > 80" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action myActionGroup \
  --severity 2

# Create log alert
az monitor scheduled-query create \
  --name "Error Log Alert" \
  --resource-group myRG \
  --scopes /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.OperationalInsights/workspaces/myWorkspace \
  --condition "count > 10" \
  --condition-query "AzureDiagnostics | where Level == 'Error'" \
  --window-size 5 \
  --evaluation-frequency 5 \
  --action-groups myActionGroup
```

---

# 5. AZURE AUTOMATION

## What It Is
Process automation using runbooks. Automate repetitive tasks, configuration management, and update management.

## Key Features
```
CAPABILITIES:
┌─────────────────────────────────────────────────────────────────┐
│ RUNBOOKS:          PowerShell or Python scripts               │
│ SCHEDULES:         Time-based execution                       │
│ WEBHOOKS:          HTTP-triggered execution                   │
│ DSC:               Desired State Configuration                │
│ UPDATE MANAGEMENT: Patch management                           │
│ CHANGE TRACKING:   Monitor config changes                     │
└─────────────────────────────────────────────────────────────────┘
```

## CLI Commands

```bash
# Create automation account
az automation account create \
  --resource-group myRG \
  --name myAutomation \
  --location eastus

# Create runbook
az automation runbook create \
  --resource-group myRG \
  --automation-account-name myAutomation \
  --name myRunbook \
  --type PowerShell

# Publish runbook
az automation runbook publish \
  --resource-group myRG \
  --automation-account-name myAutomation \
  --name myRunbook

# Start runbook
az automation runbook start \
  --resource-group myRG \
  --automation-account-name myAutomation \
  --name myRunbook
```

---

# 6. AZURE ARC

## What It Is
Extends Azure management to resources outside Azure - on-premises servers, other clouds, edge.

## What It Manages
```
ARC-ENABLED RESOURCES:
┌─────────────────────────────────────────────────────────────────┐
│ SERVERS:           Windows/Linux servers anywhere              │
│ KUBERNETES:        Any CNCF-conformant cluster                 │
│ SQL SERVER:        On-premises SQL instances                   │
│ VMWARE/SCVMM:      Private cloud VMs                           │
│ DATA SERVICES:     PostgreSQL, SQL MI at edge                  │
└─────────────────────────────────────────────────────────────────┘
```

## Benefits
```
CAPABILITIES WITH ARC:
- Azure Policy on any server
- Azure Monitor on any server
- Azure Defender on any server
- GitOps for any Kubernetes cluster
- Azure RBAC on any resource
- Inventory and tagging
```

---

# MONITORING BEST PRACTICES

```
OBSERVABILITY CHECKLIST:
┌─────────────────────────────────────────────────────────────────┐
│ ✓ Create dedicated Log Analytics workspace per environment    │
│ ✓ Enable diagnostic settings on all resources                 │
│ ✓ Use Application Insights for applications                   │
│ ✓ Set up alerts for critical metrics                          │
│ ✓ Create dashboards for team visibility                       │
│ ✓ Configure action groups for notification                    │
│ ✓ Set appropriate retention periods                           │
│ ✓ Use workbooks for detailed analysis                         │
│ ✓ Enable Container Insights for AKS                           │
│ ✓ Review and tune alerts regularly                            │
└─────────────────────────────────────────────────────────────────┘
```
