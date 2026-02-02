# Azure Compute Services - Complete Guide
## VMs, VMSS, AKS, ACI, Container Apps, Functions, App Service

---

# 1. AZURE VIRTUAL MACHINES (VMs)

## What It Is
Azure Virtual Machines provide on-demand, scalable computing resources. You get full control over the operating system, installed software, and configurations.

## When to Use
- Need full OS control
- Lift-and-shift migrations
- Running legacy applications
- Custom software requirements
- Development/test environments

## Key Concepts

### VM Sizes
```
NAMING CONVENTION: [Family][Sub-family][vCPUs][Features][Version]

Example: Standard_D4s_v5
- D = General purpose
- 4 = 4 vCPUs
- s = Premium storage capable
- v5 = Version 5

FAMILIES:
┌─────────────────────────────────────────────────────────────────┐
│ A-series: Entry-level, dev/test                                │
│ B-series: Burstable, variable workloads                        │
│ D-series: General purpose, balanced CPU/memory                 │
│ E-series: Memory optimized                                     │
│ F-series: Compute optimized                                    │
│ G-series: Memory and storage optimized                         │
│ H-series: High performance computing                           │
│ L-series: Storage optimized                                    │
│ M-series: Memory optimized (up to 4TB RAM)                     │
│ N-series: GPU enabled                                          │
└─────────────────────────────────────────────────────────────────┘
```

### Disk Types
```
MANAGED DISK TYPES:
┌──────────────────┬─────────────┬─────────────┬─────────────────┐
│ Type             │ IOPS        │ Throughput  │ Use Case        │
├──────────────────┼─────────────┼─────────────┼─────────────────┤
│ Standard HDD     │ 500         │ 60 MB/s     │ Backup, dev     │
│ Standard SSD     │ 6,000       │ 750 MB/s    │ Web servers     │
│ Premium SSD      │ 20,000      │ 900 MB/s    │ Production      │
│ Premium SSD v2   │ 80,000      │ 1,200 MB/s  │ High perf       │
│ Ultra Disk       │ 160,000     │ 4,000 MB/s  │ SAP, databases  │
└──────────────────┴─────────────┴─────────────┴─────────────────┘
```

### Availability Options
```
AVAILABILITY SETS:
- Fault Domains (FD): Physical rack isolation (max 3)
- Update Domains (UD): Logical grouping for updates (max 20)
- 99.95% SLA with 2+ VMs

AVAILABILITY ZONES:
- Physically separate datacenters within a region
- Independent power, cooling, networking
- 99.99% SLA with 2+ VMs across zones

REGIONS:
- Geographic location
- Each region has multiple Availability Zones
```

## CLI Commands

```bash
# Create a VM
az vm create \
  --resource-group myRG \
  --name myVM \
  --image Ubuntu2204 \
  --size Standard_D2s_v5 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --zone 1

# List VMs
az vm list -o table

# Start/Stop/Restart
az vm start -g myRG -n myVM
az vm stop -g myRG -n myVM
az vm restart -g myRG -n myVM

# Deallocate (stops billing for compute)
az vm deallocate -g myRG -n myVM

# Resize VM
az vm resize -g myRG -n myVM --size Standard_D4s_v5

# Get public IP
az vm show -g myRG -n myVM --show-details --query publicIps -o tsv

# Open port
az vm open-port -g myRG -n myVM --port 80

# List available sizes in region
az vm list-sizes --location eastus -o table

# List available images
az vm image list --output table
az vm image list --publisher Canonical --all --output table
```

## Best Practices
- Use Managed Disks (not unmanaged)
- Enable boot diagnostics
- Use Availability Zones for production
- Configure backup with Azure Backup
- Use Azure Bastion for secure access (no public IP)
- Apply Azure Policy for compliance
- Use Managed Identities instead of service principals
- Enable Azure Defender for security

---

# 2. VIRTUAL MACHINE SCALE SETS (VMSS)

## What It Is
VMSS lets you create and manage a group of identical, load-balanced VMs. The number of VM instances can automatically increase or decrease based on demand or a schedule.

## When to Use
- Auto-scaling applications
- High availability requirements
- Stateless applications (web servers, APIs)
- Big compute / HPC workloads
- Kubernetes nodes (AKS uses VMSS)

## Key Concepts

### Scaling Modes
```
UNIFORM MODE (Traditional):
- All VMs identical
- Managed as a group
- Best for stateless workloads

FLEXIBLE MODE (New):
- Mix VM sizes
- Use existing VMs
- More control
- Works with Availability Zones
```

### Scaling Options
```
MANUAL SCALING:
- Set instance count manually
- Good for predictable workloads

AUTOSCALE:
- Based on metrics (CPU, memory, custom)
- Schedule-based
- Predictive autoscaling (ML-based)

SCALE-IN POLICY:
- Default: Balance across zones, then FDs, then oldest
- NewestVM: Remove newest first
- OldestVM: Remove oldest first
```

## CLI Commands

```bash
# Create VMSS
az vmss create \
  --resource-group myRG \
  --name myVMSS \
  --image Ubuntu2204 \
  --vm-sku Standard_D2s_v5 \
  --instance-count 3 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --zones 1 2 3

# Scale manually
az vmss scale -g myRG -n myVMSS --new-capacity 5

# Configure autoscale
az monitor autoscale create \
  --resource-group myRG \
  --resource myVMSS \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name myAutoscale \
  --min-count 2 \
  --max-count 10 \
  --count 3

# Add autoscale rule (scale out on CPU > 70%)
az monitor autoscale rule create \
  --resource-group myRG \
  --autoscale-name myAutoscale \
  --condition "Percentage CPU > 70 avg 5m" \
  --scale out 1

# Add autoscale rule (scale in on CPU < 30%)
az monitor autoscale rule create \
  --resource-group myRG \
  --autoscale-name myAutoscale \
  --condition "Percentage CPU < 30 avg 5m" \
  --scale in 1

# Update instances (apply changes)
az vmss update-instances -g myRG -n myVMSS --instance-ids "*"

# List instances
az vmss list-instances -g myRG -n myVMSS -o table
```

---

# 3. AZURE KUBERNETES SERVICE (AKS)

## What It Is
Managed Kubernetes service. Azure handles the control plane (API server, etcd, scheduler, controller manager). You manage worker nodes.

## When to Use
- Container orchestration
- Microservices architectures
- CI/CD pipelines
- Multi-tenant applications
- Any workload that benefits from containers

## Key Concepts

### Cluster Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                     AKS CLUSTER                                 │
├─────────────────────────────────────────────────────────────────┤
│  CONTROL PLANE (Azure Managed - FREE)                          │
│  ┌─────────────┬─────────────┬─────────────┬─────────────┐     │
│  │ API Server  │ etcd        │ Scheduler   │ Controllers │     │
│  └─────────────┴─────────────┴─────────────┴─────────────┘     │
├─────────────────────────────────────────────────────────────────┤
│  NODE POOLS (You Pay)                                           │
│  ┌─────────────────────────┐  ┌─────────────────────────┐      │
│  │ System Pool (Linux)     │  │ User Pool (Linux/Win)   │      │
│  │ - CoreDNS, kube-proxy   │  │ - Your applications     │      │
│  │ - Critical addons       │  │ - Custom workloads      │      │
│  └─────────────────────────┘  └─────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘
```

### Networking Models
```
KUBENET (Basic):
- Nodes get IPs from Azure VNet
- Pods get IPs from separate range (NAT)
- Limited to 400 nodes
- Simpler, less IP consumption

AZURE CNI (Advanced):
- Nodes AND Pods get IPs from Azure VNet
- Direct pod connectivity
- Works with Network Policies
- Consumes more IPs

AZURE CNI OVERLAY:
- Nodes from VNet, Pods from overlay
- Best of both worlds
- New, recommended for large clusters
```

### Identity Options
```
CLUSTER IDENTITY:
- System-assigned managed identity (default)
- User-assigned managed identity
- Service principal (legacy)

WORKLOAD IDENTITY:
- Azure AD Workload Identity
- Pods get Azure AD tokens
- Replaces pod identity (deprecated)
```

## CLI Commands

```bash
# Create AKS cluster
az aks create \
  --resource-group myRG \
  --name myAKS \
  --node-count 3 \
  --node-vm-size Standard_D4s_v5 \
  --network-plugin azure \
  --enable-managed-identity \
  --enable-addons monitoring \
  --zones 1 2 3 \
  --generate-ssh-keys

# Get credentials (kubeconfig)
az aks get-credentials -g myRG -n myAKS

# Scale node pool
az aks scale -g myRG -n myAKS --node-count 5

# Add node pool
az aks nodepool add \
  --resource-group myRG \
  --cluster-name myAKS \
  --name gpupool \
  --node-count 2 \
  --node-vm-size Standard_NC6s_v3 \
  --node-taints sku=gpu:NoSchedule

# Enable cluster autoscaler
az aks nodepool update \
  --resource-group myRG \
  --cluster-name myAKS \
  --name nodepool1 \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 10

# Upgrade cluster
az aks get-upgrades -g myRG -n myAKS -o table
az aks upgrade -g myRG -n myAKS --kubernetes-version 1.28.0

# Enable add-ons
az aks enable-addons -g myRG -n myAKS --addons azure-keyvault-secrets-provider

# Get cluster info
az aks show -g myRG -n myAKS -o table
```

---

# 4. AZURE CONTAINER INSTANCES (ACI)

## What It Is
Serverless containers. Run containers without managing VMs or orchestrators. Per-second billing.

## When to Use
- Quick container deployment
- Batch jobs
- CI/CD agents
- Event-driven processing
- Dev/test environments
- AKS virtual node (burst)

## Key Concepts

### Container Groups
```
CONTAINER GROUP:
- One or more containers on same host
- Share lifecycle, network, storage
- Similar to Kubernetes Pod
- Single public IP for the group

┌─────────────────────────────────────────────────────────────────┐
│  CONTAINER GROUP                                                │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │ Container 1 │  │ Container 2 │  │ Container 3 │             │
│  │ (App)       │  │ (Sidecar)   │  │ (Log agent) │             │
│  └─────────────┴──┴─────────────┴──┴─────────────┘             │
│  Shared: Public IP, Volumes, vCPU/Memory                       │
└─────────────────────────────────────────────────────────────────┘
```

## CLI Commands

```bash
# Create container instance
az container create \
  --resource-group myRG \
  --name myContainer \
  --image nginx:latest \
  --cpu 1 \
  --memory 1.5 \
  --ports 80 \
  --dns-name-label myapp

# Create with environment variables
az container create \
  --resource-group myRG \
  --name myContainer \
  --image myregistry.azurecr.io/myapp:v1 \
  --cpu 2 \
  --memory 4 \
  --registry-login-server myregistry.azurecr.io \
  --registry-username $ACR_USER \
  --registry-password $ACR_PASS \
  --environment-variables KEY1=value1 KEY2=value2 \
  --secure-environment-variables SECRET1=secretvalue

# View logs
az container logs -g myRG -n myContainer

# Attach to container (stream logs)
az container attach -g myRG -n myContainer

# Execute command in container
az container exec -g myRG -n myContainer --exec-command "/bin/bash"

# Get container details
az container show -g myRG -n myContainer

# Delete container
az container delete -g myRG -n myContainer -y
```

---

# 5. AZURE CONTAINER APPS

## What It Is
Fully managed serverless container platform. Built on Kubernetes but abstracts away the complexity. Supports microservices, event-driven apps, and background jobs.

## When to Use
- Microservices
- API backends
- Event-driven processing
- Background tasks
- Web apps with autoscaling

## Key Concepts

### Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│  CONTAINER APPS ENVIRONMENT                                     │
│  (Shared boundary: VNet, Log Analytics, Dapr)                  │
│                                                                 │
│  ┌─────────────────────┐  ┌─────────────────────┐              │
│  │ Container App 1     │  │ Container App 2     │              │
│  │ - Multiple revisions│  │ - Multiple revisions│              │
│  │ - Auto-scale rules  │  │ - Auto-scale rules  │              │
│  └─────────────────────┘  └─────────────────────┘              │
└─────────────────────────────────────────────────────────────────┘
```

### Scaling
```
SCALE RULES:
- HTTP traffic (requests per second)
- CPU/Memory utilization
- Azure Queue length
- Custom (KEDA scalers)
- Scale to zero (cost savings)
```

## CLI Commands

```bash
# Create environment
az containerapp env create \
  --name myEnv \
  --resource-group myRG \
  --location eastus

# Create container app
az containerapp create \
  --name myApp \
  --resource-group myRG \
  --environment myEnv \
  --image nginx:latest \
  --target-port 80 \
  --ingress external \
  --min-replicas 0 \
  --max-replicas 10

# Update container app
az containerapp update \
  --name myApp \
  --resource-group myRG \
  --image nginx:1.25

# Create revision
az containerapp revision copy \
  --name myApp \
  --resource-group myRG \
  --image myregistry.azurecr.io/myapp:v2

# Set traffic split
az containerapp ingress traffic set \
  --name myApp \
  --resource-group myRG \
  --revision-weight myApp--v1=80 myApp--v2=20

# View logs
az containerapp logs show -n myApp -g myRG
```

---

# 6. AZURE FUNCTIONS

## What It Is
Serverless compute service for running event-driven code. Pay only for execution time.

## When to Use
- Event processing
- Scheduled tasks (cron jobs)
- API backends
- Stream processing
- IoT data processing
- Webhooks

## Key Concepts

### Hosting Plans
```
CONSUMPTION PLAN:
- True serverless
- Auto-scale (0 to many)
- 5-minute timeout (default), max 10
- Pay per execution
- Cold starts

PREMIUM PLAN:
- Pre-warmed instances (no cold start)
- VNet integration
- Unlimited execution time
- More powerful instances

DEDICATED (APP SERVICE) PLAN:
- Run on App Service
- Good for existing App Service users
- Predictable billing
```

### Triggers and Bindings
```
TRIGGERS (What starts the function):
- HTTP (REST API)
- Timer (cron)
- Blob Storage (file upload)
- Queue Storage
- Service Bus
- Event Hub
- Event Grid
- Cosmos DB (change feed)

BINDINGS (Input/Output):
- Connect to services declaratively
- No connection code needed
- Input: Read data
- Output: Write data
```

## CLI Commands

```bash
# Create Function App
az functionapp create \
  --resource-group myRG \
  --consumption-plan-location eastus \
  --runtime python \
  --runtime-version 3.10 \
  --functions-version 4 \
  --name myFunctionApp \
  --storage-account mystorageaccount

# Deploy function
func azure functionapp publish myFunctionApp

# List functions
az functionapp function list -g myRG -n myFunctionApp

# Get function URL
az functionapp function show -g myRG -n myFunctionApp --function-name myFunction

# View logs
func azure functionapp logstream myFunctionApp

# Configure app settings
az functionapp config appsettings set \
  -g myRG -n myFunctionApp \
  --settings "SETTING1=value1" "SETTING2=value2"
```

---

# 7. AZURE APP SERVICE

## What It Is
Fully managed platform for building, deploying, and scaling web apps. Supports multiple languages (.NET, Java, Node.js, Python, PHP, Ruby).

## When to Use
- Web applications
- REST APIs
- Mobile backends
- Any HTTP-based application
- When you don't want to manage infrastructure

## Key Concepts

### Service Plans
```
FREE & SHARED:
- Dev/test only
- Shared infrastructure
- No custom domains (free)
- No SLA

BASIC:
- Custom domains
- Manual scaling (3 instances max)
- 99.95% SLA

STANDARD:
- Auto-scale (10 instances)
- Deployment slots
- Daily backups

PREMIUM:
- More instances (30)
- Premium V3 hardware
- Zone redundancy
- More slots

ISOLATED (ASE):
- Dedicated environment
- VNet integration
- Highest scale (100 instances)
```

### Deployment Slots
```
DEPLOYMENT SLOTS:
- Separate instances of your app
- Each has its own URL
- Swap between slots (zero downtime)
- Test in production environment before swap

┌─────────────────────────────────────────────────────────────────┐
│  APP SERVICE                                                    │
│  ┌─────────────────────┐  ┌─────────────────────┐              │
│  │ Production Slot     │  │ Staging Slot        │              │
│  │ myapp.azurewebsites │  │ myapp-staging.azure │              │
│  │ .net                │  │ websites.net        │              │
│  └─────────────────────┘  └─────────────────────┘              │
│              ↑                      ↑                           │
│              └──────── SWAP ────────┘                           │
└─────────────────────────────────────────────────────────────────┘
```

## CLI Commands

```bash
# Create App Service Plan
az appservice plan create \
  --name myPlan \
  --resource-group myRG \
  --sku S1 \
  --is-linux

# Create Web App
az webapp create \
  --resource-group myRG \
  --plan myPlan \
  --name myWebApp \
  --runtime "NODE:18-lts"

# Deploy from Git
az webapp deployment source config \
  --resource-group myRG \
  --name myWebApp \
  --repo-url https://github.com/user/repo \
  --branch main

# Deploy from container
az webapp config container set \
  --resource-group myRG \
  --name myWebApp \
  --docker-custom-image-name myregistry.azurecr.io/myapp:v1 \
  --docker-registry-server-url https://myregistry.azurecr.io

# Create deployment slot
az webapp deployment slot create \
  --resource-group myRG \
  --name myWebApp \
  --slot staging

# Swap slots
az webapp deployment slot swap \
  --resource-group myRG \
  --name myWebApp \
  --slot staging \
  --target-slot production

# Configure app settings
az webapp config appsettings set \
  --resource-group myRG \
  --name myWebApp \
  --settings KEY1=value1 KEY2=value2

# View logs
az webapp log tail -g myRG -n myWebApp
```

---

# QUICK REFERENCE: COMPUTE SERVICES COMPARISON

| Feature | VMs | VMSS | AKS | ACI | Container Apps | Functions | App Service |
|---------|-----|------|-----|-----|----------------|-----------|-------------|
| Control | Full | Full | High | Low | Low | Minimal | Low |
| Scaling | Manual | Auto | Auto | Manual | Auto | Auto | Auto |
| Containers | No* | No* | Yes | Yes | Yes | Yes | Yes |
| Serverless | No | No | No | Yes | Yes | Yes | No |
| Best For | Legacy, custom | Scale-out | Microservices | Quick jobs | Modern apps | Events | Web apps |

*VMs can run containers but aren't container-native
