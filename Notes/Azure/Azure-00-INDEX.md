# Azure Services - Complete Reference Index
## All Major Azure Services with CLI Commands

---

## Available Guides

| # | Guide | Services Covered |
|---|-------|------------------|
| 1 | [Compute Services](./Azure-01-Compute-Services.md) | VMs, VMSS, AKS, ACI, Container Apps, Functions, App Service |
| 2 | [Networking Services](./Azure-02-Networking-Services.md) | VNet, NSG, ASG, Load Balancer, App Gateway, Front Door, Firewall, Private Link, VPN, Bastion |
| 3 | [Storage Services](./Azure-03-Storage-Services.md) | Blob, Files, Queue, Table, Managed Disks, Data Lake |
| 4 | [Database Services](./Azure-04-Database-Services.md) | SQL Database, PostgreSQL, MySQL, Cosmos DB, Redis |
| 5 | [Identity & Security](./Azure-05-Identity-Security.md) | Entra ID, Managed Identities, Key Vault, RBAC, Policy, Defender |
| 6 | [Monitoring & Management](./Azure-06-Monitoring-Management.md) | Monitor, Log Analytics, App Insights, Alerts, Automation, Arc |
| 7 | [DevOps & CI/CD](./Azure-07-DevOps-CICD.md) | Azure DevOps, Container Registry, Artifacts |
| 8 | [Messaging & Integration](./Azure-08-Messaging-Integration.md) | Service Bus, Event Hubs, Event Grid, API Management, Logic Apps |
| 9 | [Backup, DR & Governance](./Azure-09-Backup-DR-Governance.md) | Backup, Site Recovery, Cost Management, Tags, Locks, Advisor |

---

## Total Services Covered: 73

| Category | Count | Key Services |
|----------|-------|--------------|
| Compute | 9 | VMs, AKS, Functions, App Service |
| Networking | 17 | VNet, NSG, Load Balancer, Firewall |
| Storage | 7 | Blob, Files, Managed Disks |
| Databases | 6 | SQL, PostgreSQL, Cosmos DB, Redis |
| Identity | 8 | Entra ID, Key Vault, RBAC |
| Monitoring | 8 | Monitor, Log Analytics, App Insights |
| DevOps | 3 | Azure DevOps, ACR |
| Messaging | 5 | Service Bus, Event Hubs, Event Grid |
| Governance | 10 | Backup, Policy, Cost Management |

---

## Quick Reference: Essential CLI Commands

### Login & Setup
```bash
# Login
az login

# Set subscription
az account set --subscription "My Subscription"

# List subscriptions
az account list -o table

# Get current context
az account show
```

### Resource Groups
```bash
# Create
az group create --name myRG --location eastus

# List
az group list -o table

# Delete
az group delete --name myRG --yes --no-wait
```

### Common Patterns
```bash
# List resources in group
az resource list -g myRG -o table

# Get resource details
az <service> show -g myRG -n myResource

# Delete resource
az <service> delete -g myRG -n myResource
```

---

## Service Selection Guide

```
NEED TO HOST...                  →  USE
───────────────────────────────────────────────────────
Web application                  →  App Service
Containerized app                →  AKS or Container Apps
Serverless function              →  Azure Functions
Batch processing                 →  Azure Batch or ACI
Windows application              →  Virtual Machines

NEED TO STORE...
───────────────────────────────────────────────────────
Files/blobs                      →  Blob Storage
Shared files (SMB)               →  Azure Files
Relational data                  →  SQL/PostgreSQL/MySQL
NoSQL data                       →  Cosmos DB
Cache                            →  Redis

NEED TO CONNECT...
───────────────────────────────────────────────────────
Internal resources               →  VNet + Private Endpoints
Load balance HTTP                →  Application Gateway
Load balance TCP/UDP             →  Load Balancer
Global load balance              →  Front Door
On-premises                      →  VPN or ExpressRoute

NEED TO SECURE...
───────────────────────────────────────────────────────
Secrets                          →  Key Vault
Authentication                   →  Entra ID
Access control                   →  RBAC
Compliance                       →  Azure Policy
```

---

## Learning Path

**Recommended Order:**

1. **Azure Fundamentals** - Resource Groups, Subscriptions, Regions
2. **Identity** - Entra ID, RBAC, Managed Identities
3. **Networking** - VNet, NSG, Private Endpoints
4. **Compute** - VMs, App Service, AKS
5. **Storage** - Blob, Files, Managed Disks
6. **Databases** - SQL, PostgreSQL, Cosmos DB
7. **Monitoring** - Monitor, Log Analytics, Alerts
8. **DevOps** - Azure DevOps, ACR, Pipelines
9. **Security** - Key Vault, Defender, Policy
10. **Governance** - Cost Management, Tags, Backup

---

## Certification Paths

```
AZURE CERTIFICATIONS:
┌─────────────────────────────────────────────────────────────────┐
│ AZ-900:  Azure Fundamentals                                    │
│ AZ-104:  Azure Administrator                                   │
│ AZ-204:  Azure Developer                                       │
│ AZ-305:  Azure Solutions Architect                             │
│ AZ-400:  Azure DevOps Engineer                                 │
│ AZ-500:  Azure Security Engineer                               │
│ AZ-700:  Azure Network Engineer                                │
└─────────────────────────────────────────────────────────────────┘
```
