# Azure Backup, DR & Governance - Complete Guide
## Backup, Site Recovery, Cost Management, Tags, Locks, Quotas

---

# 1. AZURE BACKUP

## What It Is
Enterprise backup solution for Azure and on-premises workloads.

## What It Backs Up
```
SUPPORTED WORKLOADS:
┌─────────────────────────────────────────────────────────────────┐
│ Azure VMs:          Full VM backup                             │
│ Azure Files:        File share backup                          │
│ SQL in Azure VM:    Database backup                            │
│ SAP HANA:           Database backup                            │
│ Azure Blobs:        Blob backup                                │
│ Azure Disks:        Managed disk backup                        │
│ PostgreSQL:         Database backup                            │
│ On-premises:        Via MARS agent or MABS                     │
└─────────────────────────────────────────────────────────────────┘
```

## CLI Commands

```bash
# Create Recovery Services vault
az backup vault create \
  --resource-group myRG \
  --name myRecoveryVault \
  --location eastus

# Enable backup for VM
az backup protection enable-for-vm \
  --resource-group myRG \
  --vault-name myRecoveryVault \
  --vm myVM \
  --policy-name DefaultPolicy

# Run backup now
az backup protection backup-now \
  --resource-group myRG \
  --vault-name myRecoveryVault \
  --container-name myVM \
  --item-name myVM \
  --retain-until 30-12-2024

# List recovery points
az backup recoverypoint list \
  --resource-group myRG \
  --vault-name myRecoveryVault \
  --container-name myVM \
  --item-name myVM

# Restore VM
az backup restore restore-disks \
  --resource-group myRG \
  --vault-name myRecoveryVault \
  --container-name myVM \
  --item-name myVM \
  --rp-name <recovery-point-name> \
  --storage-account mystorageaccount
```

---

# 2. AZURE SITE RECOVERY

## What It Is
Disaster recovery as a service. Replicates VMs to secondary region for failover.

## Scenarios
```
REPLICATION SCENARIOS:
┌─────────────────────────────────────────────────────────────────┐
│ Azure to Azure:     DR within Azure                            │
│ VMware to Azure:    On-premises DR                             │
│ Hyper-V to Azure:   On-premises DR                             │
│ Physical to Azure:  Bare metal DR                              │
└─────────────────────────────────────────────────────────────────┘
```

## Key Concepts
```
SITE RECOVERY COMPONENTS:
┌─────────────────────────────────────────────────────────────────┐
│ RPO:                Recovery Point Objective (data loss)       │
│ RTO:                Recovery Time Objective (downtime)         │
│ REPLICATION:        Continuous data sync                       │
│ FAILOVER:           Switch to secondary                        │
│ FAILBACK:           Return to primary                          │
│ TEST FAILOVER:      Non-disruptive DR test                     │
└─────────────────────────────────────────────────────────────────┘
```

---

# 3. AVAILABILITY ZONES

## What It Is
Physically separate datacenters within an Azure region. Independent power, cooling, networking.

## Types of Services
```
ZONE SERVICES:
┌─────────────────────────────────────────────────────────────────┐
│ ZONAL:              Pinned to specific zone                    │
│                     VMs, Managed Disks                         │
│                                                                 │
│ ZONE-REDUNDANT:     Automatically spread across zones          │
│                     ZRS Storage, SQL DB, AKS                   │
│                                                                 │
│ NON-ZONAL:          Regional service                           │
│                     Azure AD, Traffic Manager                  │
└─────────────────────────────────────────────────────────────────┘
```

---

# 4. COST MANAGEMENT

## What It Is
Tools for monitoring, allocating, and optimizing cloud costs.

## Key Features
```
COST MANAGEMENT:
┌─────────────────────────────────────────────────────────────────┐
│ COST ANALYSIS:      View and analyze spending                  │
│ BUDGETS:            Set spending limits with alerts            │
│ RECOMMENDATIONS:    Cost optimization suggestions              │
│ EXPORTS:            Schedule cost data exports                 │
│ RESERVATIONS:       Pre-purchase for discounts                 │
└─────────────────────────────────────────────────────────────────┘
```

## CLI Commands

```bash
# Create budget
az consumption budget create \
  --budget-name myBudget \
  --amount 1000 \
  --time-grain Monthly \
  --start-date 2024-01-01 \
  --end-date 2024-12-31 \
  --resource-group myRG

# View cost
az consumption usage list \
  --start-date 2024-01-01 \
  --end-date 2024-01-31

# List reservations
az reservations reservation list
```

---

# 5. RESOURCE TAGS

## What It Is
Metadata for organizing resources. Key-value pairs for categorization, cost tracking, automation.

## Common Tag Examples
```
RECOMMENDED TAGS:
┌──────────────────┬─────────────────────────────────────────────┐
│ Environment      │ Production, Staging, Development           │
│ Owner            │ team@company.com                           │
│ CostCenter       │ CC-12345                                   │
│ Project          │ ProjectName                                │
│ Application      │ AppName                                    │
│ CreatedBy        │ terraform, manual                          │
│ ExpiresOn        │ 2024-12-31                                 │
└──────────────────┴─────────────────────────────────────────────┘
```

## CLI Commands

```bash
# Add tags to resource
az resource tag \
  --resource-group myRG \
  --name myVM \
  --resource-type Microsoft.Compute/virtualMachines \
  --tags Environment=Production Owner=team@company.com

# List resources by tag
az resource list --tag Environment=Production -o table

# Update tags on resource group
az group update --name myRG --tags Environment=Production

# Remove tag
az resource tag \
  --resource-group myRG \
  --name myVM \
  --resource-type Microsoft.Compute/virtualMachines \
  --tags Environment=Production Owner=  # Empty value removes tag
```

---

# 6. RESOURCE LOCKS

## What It Is
Prevent accidental deletion or modification of resources.

## Lock Types
```
LOCK TYPES:
┌──────────────────┬─────────────────────────────────────────────┐
│ Delete           │ Can modify, cannot delete                  │
│ ReadOnly         │ Cannot modify or delete                    │
└──────────────────┴─────────────────────────────────────────────┘
```

## CLI Commands

```bash
# Create delete lock
az lock create \
  --name CannotDelete \
  --lock-type CanNotDelete \
  --resource-group myRG \
  --resource-name myVM \
  --resource-type Microsoft.Compute/virtualMachines

# Create read-only lock on resource group
az lock create \
  --name ReadOnlyLock \
  --lock-type ReadOnly \
  --resource-group myRG

# List locks
az lock list --resource-group myRG

# Delete lock
az lock delete --name CannotDelete --resource-group myRG
```

---

# 7. QUOTAS AND LIMITS

## What It Is
Service limits that cap resource usage. Some are hard limits, others can be increased.

## Common Limits
```
DEFAULT QUOTAS (Examples):
┌─────────────────────────────────────────────────────────────────┐
│ VMs per region:           25,000                               │
│ vCPUs per region:         Varies by subscription type          │
│ Storage accounts:         250 per region                       │
│ VNets per region:         1,000                                │
│ NSGs per region:          5,000                                │
│ Load balancers:           1,000                                │
│ Public IPs:               Varies                               │
└─────────────────────────────────────────────────────────────────┘
```

## CLI Commands

```bash
# View compute quotas
az vm list-usage --location eastus -o table

# View network quotas
az network list-usages --location eastus -o table

# Request quota increase
# Done via Azure Portal support request
```

---

# 8. AZURE ADVISOR

## What It Is
Personalized consultant for Azure best practices. Provides recommendations for cost, security, reliability, performance.

## Categories
```
ADVISOR CATEGORIES:
┌─────────────────────────────────────────────────────────────────┐
│ COST:               Right-size VMs, unused resources           │
│ SECURITY:           Enable MFA, secure endpoints               │
│ RELIABILITY:        Enable backup, zone redundancy             │
│ PERFORMANCE:        Upgrade SKUs, optimize queries             │
│ OPERATIONAL EXCELLENCE: Improve processes                      │
└─────────────────────────────────────────────────────────────────┘
```

## CLI Commands

```bash
# Get recommendations
az advisor recommendation list -o table

# Get cost recommendations
az advisor recommendation list --category Cost -o table
```

---

# GOVERNANCE BEST PRACTICES

```
GOVERNANCE CHECKLIST:
┌─────────────────────────────────────────────────────────────────┐
│ ✓ Implement naming convention                                  │
│ ✓ Use resource tags consistently                               │
│ ✓ Apply resource locks on critical resources                   │
│ ✓ Set up budgets and alerts                                    │
│ ✓ Use Azure Policy for compliance                              │
│ ✓ Organize with Management Groups                              │
│ ✓ Review Advisor recommendations regularly                     │
│ ✓ Document architecture and processes                          │
│ ✓ Monitor quota usage                                          │
│ ✓ Plan for disaster recovery                                   │
└─────────────────────────────────────────────────────────────────┘
```
