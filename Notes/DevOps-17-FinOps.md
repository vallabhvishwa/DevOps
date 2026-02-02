# DevOps Engineer's Complete Reference Guide
# Part 17: FinOps & Cost Optimization

---

## Table of Contents

1. [Introduction to FinOps](#1-introduction-to-finops)
2. [FinOps Principles](#2-finops-principles)
3. [Azure Cost Management](#3-azure-cost-management)
4. [Kubernetes Cost Optimization](#4-kubernetes-cost-optimization)
5. [Right-Sizing Resources](#5-right-sizing-resources)
6. [Reserved Instances and Savings](#6-reserved-instances-and-savings)
7. [Cost Allocation and Chargeback](#7-cost-allocation-and-chargeback)
8. [Automation for Cost Control](#8-automation-for-cost-control)
9. [FinOps Best Practices](#9-finops-best-practices)

---

## 1. Introduction to FinOps

### 1.1 What is FinOps?

```
FinOps Definition:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  FinOps = Financial Operations for Cloud                       │
│                                                                 │
│  "FinOps is the practice of bringing financial accountability  │
│   to the variable spend model of cloud, enabling distributed   │
│   teams to make business trade-offs between speed, cost, and   │
│   quality."                                                     │
│                                    - FinOps Foundation          │
│                                                                 │
│  Key Stakeholders:                                              │
│  ├── Engineering: Optimizes resource usage                      │
│  ├── Finance: Manages budgets and forecasting                   │
│  ├── Business: Understands cost/value trade-offs                │
│  └── FinOps Team: Facilitates collaboration                     │
│                                                                 │
│  Why FinOps Matters:                                            │
│  ├── Cloud spend grows 20-30% YoY                               │
│  ├── 30%+ of cloud spend is wasted                              │
│  ├── Variable costs require active management                   │
│  └── Engineers make spending decisions daily                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 FinOps Lifecycle

```
FinOps Lifecycle:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│                    ┌──────────────┐                             │
│                    │   INFORM     │                             │
│                    │ Visibility   │                             │
│                    │ Allocation   │                             │
│                    │ Benchmarking │                             │
│                    └──────┬───────┘                             │
│                           │                                     │
│         ┌─────────────────┴─────────────────┐                   │
│         │                                   │                   │
│         ▼                                   ▼                   │
│  ┌──────────────┐                   ┌──────────────┐            │
│  │   OPTIMIZE   │                   │   OPERATE    │            │
│  │ Right-sizing │◄─────────────────►│ Governance   │            │
│  │ Reservations │                   │ Automation   │            │
│  │ Spot/Preempt │                   │ Policies     │            │
│  └──────────────┘                   └──────────────┘            │
│                                                                 │
│  Continuous Cycle:                                              │
│  1. INFORM: See and understand costs                            │
│  2. OPTIMIZE: Reduce waste, maximize efficiency                 │
│  3. OPERATE: Embed FinOps in processes                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. FinOps Principles

```
FinOps Principles:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. TEAMS NEED TO COLLABORATE                                   │
│     ├── Finance, Engineering, Business work together            │
│     ├── Shared accountability for cloud costs                   │
│     └── Common language and metrics                             │
│                                                                 │
│  2. EVERYONE TAKES OWNERSHIP                                    │
│     ├── Engineers own their resource costs                      │
│     ├── Decentralized decision-making                           │
│     └── Cost as a first-class metric                            │
│                                                                 │
│  3. CENTRALIZED FINOPS TEAM DRIVES EFFICIENCY                   │
│     ├── FinOps team enables, doesn't gate                       │
│     ├── Provides tools, training, best practices                │
│     └── Negotiates rates, manages commitments                   │
│                                                                 │
│  4. TIMELY REPORTS ACCESSIBLE TO ALL                            │
│     ├── Near real-time cost visibility                          │
│     ├── Self-service dashboards                                 │
│     └── Proactive anomaly alerts                                │
│                                                                 │
│  5. BUSINESS VALUE OF CLOUD DRIVES DECISIONS                    │
│     ├── Cost per customer, per transaction                      │
│     ├── Unit economics matter                                   │
│     └── Trade-offs between speed, cost, quality                 │
│                                                                 │
│  6. VARIABLE COST MODEL IS A FEATURE                            │
│     ├── Use elasticity to your advantage                        │
│     ├── Right-size continuously                                 │
│     └── Pay only for what you use                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Azure Cost Management

### 3.1 Cost Analysis

```bash
# Azure CLI cost commands
az consumption usage list --start-date 2024-01-01 --end-date 2024-01-31

# Get current budget status
az consumption budget list --resource-group myRG

# Create budget with alert
az consumption budget create \
  --budget-name monthly-limit \
  --amount 1000 \
  --time-grain Monthly \
  --start-date 2024-01-01 \
  --end-date 2024-12-31 \
  --resource-group myRG \
  --notifications-enabled true \
  --notification-key alert80 \
  --notification-threshold 80 \
  --notification-emails admin@company.com

# Export cost data
az costmanagement export create \
  --name daily-export \
  --type Usage \
  --scope "/subscriptions/{subscription-id}" \
  --storage-account-id "/subscriptions/.../storageAccounts/exports" \
  --storage-container exports \
  --recurrence Daily
```

### 3.2 Cost Queries (Azure Resource Graph)

```kusto
// Resources without tags (wasted money risk)
Resources
| where tags == "{}" or isnull(tags)
| project name, type, resourceGroup, subscriptionId

// Expensive resources by type
Resources
| summarize count() by type
| order by count_ desc

// Unused disks
Resources
| where type == "microsoft.compute/disks"
| where properties.diskState == "Unattached"
| project name, resourceGroup, properties.diskSizeGB

// VMs by size (right-sizing candidates)
Resources
| where type == "microsoft.compute/virtualmachines"
| extend vmSize = properties.hardwareProfile.vmSize
| summarize count() by tostring(vmSize)
| order by count_ desc
```

### 3.3 Azure Advisor Recommendations

```bash
# Get cost recommendations
az advisor recommendation list --category Cost

# Example recommendations:
# - Right-size or shutdown underutilized VMs
# - Buy Reserved Instances for steady workloads
# - Delete unattached disks
# - Use Azure Hybrid Benefit
# - Optimize App Service plans
```

---

## 4. Kubernetes Cost Optimization

### 4.1 Understanding K8s Costs

```
Kubernetes Cost Breakdown:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  INFRASTRUCTURE COSTS:                                          │
│  ├── Control plane (managed: AKS free, EKS $0.10/hr)            │
│  ├── Node VMs (largest cost component)                          │
│  ├── Storage (PVs, managed disks)                               │
│  ├── Networking (LB, egress, bandwidth)                         │
│  └── Container registry                                         │
│                                                                 │
│  HIDDEN COSTS:                                                  │
│  ├── Over-provisioned pods (requests > usage)                   │
│  ├── Idle nodes (not enough pods to fill)                       │
│  ├── Cross-AZ traffic                                           │
│  ├── Logging and monitoring ingestion                           │
│  └── Undeleted test resources                                   │
│                                                                 │
│  COST VISIBILITY TOOLS:                                         │
│  ├── Kubecost (open source + commercial)                        │
│  ├── OpenCost (CNCF project)                                    │
│  ├── CloudHealth                                                │
│  └── Spot.io                                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 Kubecost/OpenCost

```bash
# Install OpenCost
helm install opencost --repo https://opencost.github.io/opencost-helm-chart opencost \
  --namespace opencost-system --create-namespace

# Install Kubecost
helm install kubecost cost-analyzer \
  --repo https://kubecost.github.io/cost-analyzer/ \
  --namespace kubecost --create-namespace

# Access Kubecost UI
kubectl port-forward -n kubecost svc/kubecost-cost-analyzer 9090:9090
```

### 4.3 Resource Optimization

```yaml
# Properly sized pod
apiVersion: v1
kind: Pod
metadata:
  name: optimized-app
spec:
  containers:
  - name: app
    image: myapp:latest
    resources:
      requests:           # Scheduling decisions
        memory: "256Mi"   # Based on actual usage
        cpu: "100m"       # P95 of historical usage
      limits:
        memory: "512Mi"   # Prevents OOM kill of node
        cpu: "500m"       # Optional: can cause throttling

---
# Vertical Pod Autoscaler (right-sizing recommendations)
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: myapp-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  updatePolicy:
    updateMode: "Auto"  # Off, Initial, Recreate, Auto
  resourcePolicy:
    containerPolicies:
    - containerName: '*'
      minAllowed:
        cpu: 50m
        memory: 128Mi
      maxAllowed:
        cpu: 1
        memory: 1Gi
```

### 4.4 Node Optimization

```yaml
# Use Spot instances for fault-tolerant workloads
apiVersion: apps/v1
kind: Deployment
metadata:
  name: batch-processor
spec:
  template:
    spec:
      nodeSelector:
        kubernetes.azure.com/scalesetpriority: spot
      tolerations:
      - key: kubernetes.azure.com/scalesetpriority
        operator: Equal
        value: spot
        effect: NoSchedule
      containers:
      - name: processor
        image: batch:latest

---
# Pod topology spread for efficient node usage
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: ScheduleAnyway
        labelSelector:
          matchLabels:
            app: myapp
```

---

## 5. Right-Sizing Resources

### 5.1 Right-Sizing Process

```
Right-Sizing Workflow:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. COLLECT METRICS (2+ weeks)                                  │
│     ├── CPU utilization (avg, P95, max)                         │
│     ├── Memory utilization                                      │
│     ├── Network I/O                                             │
│     └── Disk IOPS                                               │
│                                                                 │
│  2. ANALYZE PATTERNS                                            │
│     ├── Peak hours vs off-hours                                 │
│     ├── Weekday vs weekend                                      │
│     ├── Seasonal patterns                                       │
│     └── One-time spikes                                         │
│                                                                 │
│  3. IDENTIFY CANDIDATES                                         │
│     ├── Underutilized (<20% CPU for 2+ weeks)                   │
│     ├── Over-provisioned (high headroom)                        │
│     └── Idle resources                                          │
│                                                                 │
│  4. IMPLEMENT CHANGES                                           │
│     ├── Test in staging first                                   │
│     ├── Change one size at a time                               │
│     ├── Monitor after change                                    │
│     └── Document savings                                        │
│                                                                 │
│  5. AUTOMATE                                                    │
│     ├── VPA for pods                                            │
│     ├── HPA for scaling                                         │
│     └── Scheduled scaling                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 Prometheus Queries for Right-Sizing

```promql
# Container CPU usage vs requests
sum(rate(container_cpu_usage_seconds_total{container!=""}[5m])) by (pod)
/
sum(kube_pod_container_resource_requests{resource="cpu"}) by (pod)

# Over-provisioned pods (using < 50% of requests)
sum(rate(container_cpu_usage_seconds_total{container!=""}[24h])) by (pod)
/
sum(kube_pod_container_resource_requests{resource="cpu"}) by (pod)
< 0.5

# Memory efficiency
sum(container_memory_working_set_bytes{container!=""}) by (pod)
/
sum(kube_pod_container_resource_requests{resource="memory"}) by (pod)

# Node CPU utilization
(1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) by (node)) * 100

# Unschedulable capacity (wasted)
sum(kube_node_status_allocatable{resource="cpu"}) by (node)
-
sum(kube_pod_container_resource_requests{resource="cpu"}) by (node)
```

---

## 6. Reserved Instances and Savings

### 6.1 Azure Savings Options

```
Azure Savings Options:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  RESERVED INSTANCES (RI):                                       │
│  ├── 1-year: ~20-30% savings                                    │
│  ├── 3-year: ~40-60% savings                                    │
│  ├── Scope: Subscription, Resource Group, or Shared             │
│  └── Best for: Steady-state workloads                           │
│                                                                 │
│  AZURE SAVINGS PLAN:                                            │
│  ├── Hourly compute commitment                                  │
│  ├── Flexible across VM sizes and regions                       │
│  ├── 1-year or 3-year terms                                     │
│  └── Best for: Variable workloads                               │
│                                                                 │
│  SPOT VMS:                                                      │
│  ├── Up to 90% savings                                          │
│  ├── Can be evicted with 30s notice                             │
│  ├── Best for: Batch, CI/CD, fault-tolerant workloads           │
│  └── Not for: Production stateful workloads                     │
│                                                                 │
│  AZURE HYBRID BENEFIT:                                          │
│  ├── Use existing Windows Server/SQL licenses                   │
│  ├── Up to 40% savings on Windows VMs                           │
│  └── Up to 55% savings on SQL                                   │
│                                                                 │
│  DEV/TEST PRICING:                                              │
│  ├── Reduced rates for dev/test subscriptions                   │
│  └── No Windows license costs                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 6.2 Reservation Planning

```bash
# Get RI recommendations
az consumption reservation-recommendation list \
  --scope "subscriptions/{subscription-id}" \
  --look-back-period Last30Days

# Purchase RI
az reservations reservation-order purchase \
  --reservation-order-id {order-id} \
  --sku Standard_D4s_v3 \
  --location eastus \
  --reserved-resource-type VirtualMachines \
  --billing-scope-id "/subscriptions/{sub-id}" \
  --term P1Y \
  --quantity 10 \
  --applied-scope-type Shared
```

---

## 7. Cost Allocation and Chargeback

### 7.1 Tagging Strategy

```
Tagging Requirements:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  MANDATORY TAGS:                                                │
│  ├── Environment: dev, staging, prod                            │
│  ├── Team/Owner: platform, payments, frontend                   │
│  ├── CostCenter: CC-12345                                       │
│  ├── Project: project-alpha                                     │
│  └── ManagedBy: terraform, manual                               │
│                                                                 │
│  OPTIONAL TAGS:                                                 │
│  ├── Application: myapp, api-gateway                            │
│  ├── Version: v1.0.0                                            │
│  ├── CreatedDate: 2024-01-15                                    │
│  └── ExpiryDate: 2024-03-15 (for temp resources)                │
│                                                                 │
│  ENFORCEMENT:                                                   │
│  ├── Azure Policy: Require tags on resource creation            │
│  ├── Terraform: Variable validation                             │
│  └── CI/CD: Block deployment without tags                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```hcl
# Terraform: Enforce tagging
variable "required_tags" {
  description = "Required tags for all resources"
  type = object({
    environment = string
    team        = string
    cost_center = string
  })
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.required_tags.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

resource "azurerm_resource_group" "example" {
  name     = "rg-${var.required_tags.team}-${var.required_tags.environment}"
  location = var.location
  
  tags = {
    Environment = var.required_tags.environment
    Team        = var.required_tags.team
    CostCenter  = var.required_tags.cost_center
    ManagedBy   = "Terraform"
  }
}
```

### 7.2 Kubernetes Labels for Cost Allocation

```yaml
# Pod with cost labels
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    app: myapp
    team: payments        # For cost allocation
    environment: prod     # Environment
    cost-center: CC-12345 # Chargeback

---
# Namespace with cost metadata
apiVersion: v1
kind: Namespace
metadata:
  name: payments
  labels:
    team: payments
    cost-center: CC-12345
  annotations:
    budget: "5000"         # Monthly budget in USD
    owner: payments@co.com # Contact
```

---

## 8. Automation for Cost Control

### 8.1 Scheduled Scaling

```yaml
# KEDA ScaledObject for scheduled scaling
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: scheduled-scaling
spec:
  scaleTargetRef:
    name: myapp
  minReplicaCount: 1
  maxReplicaCount: 10
  triggers:
  # Business hours (more replicas)
  - type: cron
    metadata:
      timezone: America/New_York
      start: "0 8 * * 1-5"    # 8 AM Mon-Fri
      end: "0 18 * * 1-5"     # 6 PM Mon-Fri
      desiredReplicas: "5"
  # Off-hours (fewer replicas)
  - type: cron
    metadata:
      timezone: America/New_York
      start: "0 18 * * 1-5"
      end: "0 8 * * 1-5"
      desiredReplicas: "1"

---
# Azure Automation for VM start/stop
# (Configured in Azure Automation Account)
# Schedule: Stop VMs at 7 PM, Start at 7 AM
```

### 8.2 Automatic Cleanup

```python
#!/usr/bin/env python3
"""Clean up unused Azure resources."""
from azure.identity import DefaultAzureCredential
from azure.mgmt.compute import ComputeManagementClient
from azure.mgmt.resource import ResourceManagementClient
from datetime import datetime, timedelta

credential = DefaultAzureCredential()
subscription_id = "your-subscription-id"

compute_client = ComputeManagementClient(credential, subscription_id)
resource_client = ResourceManagementClient(credential, subscription_id)

def cleanup_unattached_disks(dry_run=True):
    """Delete disks that have been unattached for 30+ days."""
    deleted = 0
    for disk in compute_client.disks.list():
        if disk.disk_state == "Unattached":
            # Check if unattached for 30+ days
            if disk.time_created < datetime.now(disk.time_created.tzinfo) - timedelta(days=30):
                print(f"{'Would delete' if dry_run else 'Deleting'}: {disk.name}")
                if not dry_run:
                    compute_client.disks.begin_delete(
                        disk.id.split('/')[4],  # resource group
                        disk.name
                    )
                deleted += 1
    return deleted

def cleanup_expired_resources(dry_run=True):
    """Delete resources with passed expiry date."""
    for resource in resource_client.resources.list():
        tags = resource.tags or {}
        expiry = tags.get("ExpiryDate")
        if expiry:
            expiry_date = datetime.strptime(expiry, "%Y-%m-%d")
            if expiry_date < datetime.now():
                print(f"Expired: {resource.name} (expired {expiry})")
                if not dry_run:
                    resource_client.resources.begin_delete_by_id(
                        resource.id,
                        "2021-04-01"
                    )

if __name__ == "__main__":
    print("=== Cleanup Report ===")
    disks_deleted = cleanup_unattached_disks(dry_run=True)
    print(f"Unattached disks to delete: {disks_deleted}")
    cleanup_expired_resources(dry_run=True)
```

---

## 9. FinOps Best Practices

```
FinOps Best Practices Summary:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  VISIBILITY:                                                    │
│  ├── Tag 100% of resources                                      │
│  ├── Set up cost dashboards by team/project                     │
│  ├── Configure daily/weekly cost emails                         │
│  └── Enable anomaly detection alerts                            │
│                                                                 │
│  OPTIMIZATION:                                                  │
│  ├── Right-size based on actual usage (not guesses)             │
│  ├── Use Spot/Preemptible for suitable workloads                │
│  ├── Buy Reserved Instances for steady-state                    │
│  ├── Delete unused resources (disks, IPs, snapshots)            │
│  ├── Implement scheduled scaling                                │
│  └── Use appropriate storage tiers                              │
│                                                                 │
│  GOVERNANCE:                                                    │
│  ├── Set budgets with alerts (50%, 80%, 100%)                   │
│  ├── Enforce tagging via policies                               │
│  ├── Implement approval for large resources                     │
│  ├── Regular cost reviews (weekly/monthly)                      │
│  └── Define cost ownership by team                              │
│                                                                 │
│  KUBERNETES SPECIFIC:                                           │
│  ├── Set requests and limits on all pods                        │
│  ├── Use VPA for right-sizing recommendations                   │
│  ├── Use HPA for demand-based scaling                           │
│  ├── Cluster autoscaler for node efficiency                     │
│  ├── Use Spot node pools for fault-tolerant workloads           │
│  └── Monitor with Kubecost/OpenCost                             │
│                                                                 │
│  CULTURE:                                                       │
│  ├── Include cost in PR reviews                                 │
│  ├── Celebrate cost savings                                     │
│  ├── Make cost a deployment metric                              │
│  └── Train engineers on cloud economics                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

Quick Wins (Do This Week):
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. Find and delete unattached disks            (Immediate $$$) │
│  2. Stop dev/test VMs outside work hours        (30-50% savings)│
│  3. Enable Azure Advisor cost recommendations   (Visibility)    │
│  4. Set up budget alerts                        (Prevention)    │
│  5. Identify Reserved Instance candidates       (20-60% savings)│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Summary

FinOps enables cloud cost management:

1. **Lifecycle**: Inform → Optimize → Operate (continuous)
2. **Visibility**: Tagging, dashboards, cost allocation
3. **Optimization**: Right-sizing, reservations, spot instances
4. **Kubernetes**: Resource requests/limits, VPA, HPA, Kubecost
5. **Automation**: Scheduled scaling, cleanup scripts
6. **Governance**: Budgets, policies, chargeback

Cost optimization is an ongoing practice, not a one-time project. Embed cost awareness into engineering culture.

---

## Complete Guide Index

This completes the DevOps Engineer's Complete Reference Guide. The full series includes:

| Part | Topic | Focus |
|------|-------|-------|
| 1-2 | Linux | Fundamentals and Advanced |
| 3 | Networking | OSI, TCP/IP, DNS, Firewalls |
| 4 | Kubernetes | Architecture, Objects, kubectl |
| 5 | AKS & Azure | Azure services, CLI |
| 6 | Jenkins | CI/CD pipelines |
| 7 | Observability | Prometheus, Logging, Tracing |
| 8 | Terraform | Infrastructure as Code |
| 9 | System Design | Architecture patterns |
| 10 | SRE | SLOs, Error Budgets, Incidents |
| 11 | GitOps | ArgoCD, Flux |
| 12 | Service Mesh | Istio |
| 13 | Security | Zero Trust, Container Security |
| 14 | Chaos Engineering | Chaos Mesh, Litmus |
| 15 | Programming | Python & Go |
| 16 | Multi-Cloud | AWS & GCP basics |
| 17 | FinOps | Cost Optimization |
