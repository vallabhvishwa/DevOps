# The Complete DevOps Engineer's Reference Guide
## Part 5: AKS & Azure Services for DevOps

---

# Chapter 15: Azure Kubernetes Service (AKS)

## 15.1 AKS Architecture

```
AKS Architecture:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│                    Azure Managed Control Plane                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  (Microsoft manages this - you don't pay for it)            │   │
│  │                                                             │   │
│  │  ┌───────────┐ ┌──────┐ ┌──────────┐ ┌────────────────┐    │   │
│  │  │ API Server│ │ etcd │ │Scheduler │ │Controller Mgr  │    │   │
│  │  └───────────┘ └──────┘ └──────────┘ └────────────────┘    │   │
│  │                                                             │   │
│  │  SLA: 99.5% (free), 99.9% (Standard), 99.95% (Premium)     │   │
│  │                                                             │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│                              │                                      │
│                              ▼                                      │
│                    Customer Managed Nodes                           │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  (You pay for these - Azure VMs in your subscription)      │   │
│  │                                                             │   │
│  │  ┌──────────────────────────────────────────────────────┐  │   │
│  │  │  System Node Pool (required)                          │  │   │
│  │  │  - Runs critical system components                    │  │   │
│  │  │  - CoreDNS, metrics-server, kube-proxy               │  │   │
│  │  │  - Mode: System                                       │  │   │
│  │  │  - Taint: CriticalAddonsOnly=true:NoSchedule         │  │   │
│  │  └──────────────────────────────────────────────────────┘  │   │
│  │                                                             │   │
│  │  ┌──────────────────────────────────────────────────────┐  │   │
│  │  │  User Node Pool(s) (optional)                         │  │   │
│  │  │  - Runs your workloads                                │  │   │
│  │  │  - Can have multiple pools                            │  │   │
│  │  │  - Different VM sizes, autoscaling                    │  │   │
│  │  │  - Spot instances supported                           │  │   │
│  │  └──────────────────────────────────────────────────────┘  │   │
│  │                                                             │   │
│  │  Underlying: VM Scale Sets (VMSS)                          │   │
│  │                                                             │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  Associated Resources (created with AKS):                           │
│  ├── Virtual Network (VNet) - or use existing                      │
│  ├── Load Balancer (Standard)                                      │
│  ├── Public IP (for load balancer)                                 │
│  ├── Managed Identity                                              │
│  └── MC_* Resource Group (managed)                                 │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 15.2 AKS Networking Deep Dive

### Network Models

```
AKS Network Models:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  1. kubenet (Basic Networking)                                      │
│  ├── Nodes get VNet IP, pods get IPs from different range         │
│  ├── Pods use NAT to communicate outside node                      │
│  ├── Simple, fewer IP addresses needed                             │
│  ├── Azure resources can't directly reach pods                     │
│  ├── User-defined routes required                                  │
│  └── Limited to 400 nodes per cluster                              │
│                                                                     │
│     VNet: 10.0.0.0/16                                              │
│     Node Subnet: 10.0.0.0/24 (nodes get these IPs)                 │
│     Pod CIDR: 10.244.0.0/16 (pods get these, not in VNet)          │
│                                                                     │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                     │
│  2. Azure CNI (Advanced Networking)                                 │
│  ├── Every pod gets a VNet IP address                              │
│  ├── Pods directly reachable from VNet                             │
│  ├── No NAT needed                                                 │
│  ├── Requires more IP addresses (plan carefully!)                  │
│  ├── Supports Network Policies                                      │
│  └── Required for Windows node pools                               │
│                                                                     │
│     VNet: 10.0.0.0/8                                               │
│     AKS Subnet: 10.1.0.0/16                                        │
│     Each node uses ~30 IPs (1 node + 30 pods)                      │
│     100 nodes × 30 = 3,000 IPs needed                              │
│                                                                     │
│  IP Calculation for Azure CNI:                                      │
│  IPs needed = (nodes × max_pods_per_node) + nodes + reserved       │
│  Example: 50 nodes × 30 pods = 1,500 + 50 + 5 = 1,555 IPs          │
│  Minimum subnet: /21 (2,046 IPs)                                   │
│                                                                     │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                     │
│  3. Azure CNI Overlay (Newer Option)                                │
│  ├── Pods get IPs from overlay network (not VNet)                  │
│  ├── Nodes get VNet IPs                                            │
│  ├── Fewer VNet IPs needed (like kubenet)                          │
│  ├── But pods can still use Network Policies                       │
│  └── Best of both worlds                                           │
│                                                                     │
│  4. Bring Your Own CNI                                              │
│  ├── Use Cilium, Calico, etc.                                      │
│  └── Full control over networking                                  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Load Balancing

```yaml
# Standard Load Balancer (default)
apiVersion: v1
kind: Service
metadata:
  name: my-service
  annotations:
    # Internal load balancer
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"
    
    # Specific subnet for internal LB
    service.beta.kubernetes.io/azure-load-balancer-internal-subnet: "internal-lb-subnet"
    
    # Health probe settings
    service.beta.kubernetes.io/azure-load-balancer-health-probe-protocol: "tcp"
    service.beta.kubernetes.io/azure-load-balancer-health-probe-request-path: "/health"
    
    # Resource group for LB
    service.beta.kubernetes.io/azure-load-balancer-resource-group: "my-rg"
    
    # Static IP
    service.beta.kubernetes.io/azure-pip-name: "my-static-pip"
spec:
  type: LoadBalancer
  loadBalancerIP: 10.0.0.100          # Request specific IP
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: myapp
```

## 15.3 AKS Identity and Security

### Managed Identities

```
AKS Identity Types:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  1. Cluster Identity (Control Plane)                                │
│  ├── Used by AKS to manage Azure resources                         │
│  ├── Creates/manages load balancers, disks, IPs                    │
│  ├── Options:                                                       │
│  │   - System-assigned managed identity (default)                  │
│  │   - User-assigned managed identity                              │
│  │   - Service principal (legacy, not recommended)                 │
│  └── Assigned at cluster creation                                  │
│                                                                     │
│  2. Kubelet Identity                                                │
│  ├── Used by kubelet on nodes                                       │
│  ├── Pulls images from ACR                                          │
│  └── Can be separate from cluster identity                         │
│                                                                     │
│  3. Workload Identity (Pod-level)                                   │
│  ├── Pods can authenticate to Azure services                       │
│  ├── Uses Kubernetes ServiceAccount + Azure AD                     │
│  ├── Federation between K8s and Azure AD                           │
│  └── Replaces AAD Pod Identity (deprecated)                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Workload Identity Setup

```yaml
# 1. Create Azure User-Assigned Managed Identity
# az identity create --name myapp-identity --resource-group myRG

# 2. Create Kubernetes ServiceAccount with annotation
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myapp-sa
  namespace: default
  annotations:
    azure.workload.identity/client-id: "<managed-identity-client-id>"
---
# 3. Create federated credential (Azure CLI)
# az identity federated-credential create \
#   --name myapp-federated-cred \
#   --identity-name myapp-identity \
#   --resource-group myRG \
#   --issuer "<AKS-OIDC-issuer-URL>" \
#   --subject system:serviceaccount:default:myapp-sa

# 4. Use in Pod
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    azure.workload.identity/use: "true"
spec:
  serviceAccountName: myapp-sa
  containers:
  - name: myapp
    image: myapp:latest
    env:
    - name: AZURE_CLIENT_ID
      value: "<managed-identity-client-id>"
    # Azure SDK will automatically use workload identity
```

### Azure Key Vault Integration

```yaml
# Using Secrets Store CSI Driver
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: azure-keyvault-secrets
spec:
  provider: azure
  parameters:
    usePodIdentity: "false"
    useVMManagedIdentity: "true"       # For workload identity
    userAssignedIdentityID: "<managed-identity-client-id>"
    keyvaultName: "mykeyvault"
    tenantId: "<tenant-id>"
    objects: |
      array:
        - |
          objectName: database-password
          objectType: secret
        - |
          objectName: api-key
          objectType: secret
        - |
          objectName: tls-cert
          objectType: cert
  secretObjects:                        # Sync to Kubernetes secrets
  - secretName: myapp-secrets
    type: Opaque
    data:
    - objectName: database-password
      key: DB_PASSWORD
    - objectName: api-key
      key: API_KEY
---
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: myapp
    image: myapp:latest
    volumeMounts:
    - name: secrets-store
      mountPath: "/mnt/secrets-store"
      readOnly: true
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: myapp-secrets
          key: DB_PASSWORD
  volumes:
  - name: secrets-store
    csi:
      driver: secrets-store.csi.k8s.io
      readOnly: true
      volumeAttributes:
        secretProviderClass: azure-keyvault-secrets
```

## 15.4 AKS Storage

```yaml
# Azure Disk StorageClass (default)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-premium-retain
provisioner: disk.csi.azure.com
parameters:
  skuName: Premium_LRS                # Standard_LRS, StandardSSD_LRS, Premium_LRS, UltraSSD_LRS
  cachingmode: ReadOnly               # None, ReadOnly, ReadWrite
  # For specific zones
  # location: eastus
  # storageaccounttype: Premium_LRS
reclaimPolicy: Retain                 # Default is Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer  # Immediate or WaitForFirstConsumer
---
# Azure Files StorageClass (for RWX)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: azurefile-premium
provisioner: file.csi.azure.com
parameters:
  skuName: Premium_LRS
  # For private endpoints
  # resourceGroup: myRG
  # storageAccount: mystorageaccount
  # shareName: myshare
mountOptions:
  - dir_mode=0777
  - file_mode=0777
  - uid=0
  - gid=0
  - mfsymlinks
  - cache=strict
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: Immediate
```

## 15.5 AKS Monitoring and Diagnostics

```bash
# Enable Container Insights
az aks enable-addons \
  --resource-group myRG \
  --name myAKS \
  --addons monitoring \
  --workspace-resource-id /subscriptions/.../workspaces/myLA

# Check AKS cluster health
az aks show --resource-group myRG --name myAKS --query "provisioningState"
az aks show --resource-group myRG --name myAKS --query "powerState"

# Get diagnostic logs
az aks get-credentials --resource-group myRG --name myAKS
kubectl get events --sort-by='.lastTimestamp' -A

# Enable diagnostic settings
az monitor diagnostic-settings create \
  --resource /subscriptions/.../resourceGroups/myRG/providers/Microsoft.ContainerService/managedClusters/myAKS \
  --name aks-diagnostics \
  --workspace /subscriptions/.../workspaces/myLA \
  --logs '[
    {"category": "kube-apiserver", "enabled": true},
    {"category": "kube-controller-manager", "enabled": true},
    {"category": "kube-scheduler", "enabled": true},
    {"category": "kube-audit", "enabled": true},
    {"category": "cluster-autoscaler", "enabled": true}
  ]'
```

## 15.6 AKS Operations

### Cluster Upgrades

```bash
# Check available upgrades
az aks get-upgrades --resource-group myRG --name myAKS --output table

# Upgrade cluster (control plane + all node pools)
az aks upgrade --resource-group myRG --name myAKS --kubernetes-version 1.28.0

# Upgrade control plane only
az aks upgrade --resource-group myRG --name myAKS --kubernetes-version 1.28.0 --control-plane-only

# Upgrade specific node pool
az aks nodepool upgrade --resource-group myRG --cluster-name myAKS --name nodepool1 --kubernetes-version 1.28.0

# Node image upgrade (without K8s version change)
az aks nodepool upgrade --resource-group myRG --cluster-name myAKS --name nodepool1 --node-image-only
```

### Node Pool Management

```bash
# List node pools
az aks nodepool list --resource-group myRG --cluster-name myAKS --output table

# Add node pool
az aks nodepool add \
  --resource-group myRG \
  --cluster-name myAKS \
  --name userpool \
  --node-count 3 \
  --node-vm-size Standard_DS2_v2 \
  --mode User \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 10 \
  --node-taints "workload=app:NoSchedule" \
  --labels environment=production app=myapp

# Scale node pool
az aks nodepool scale --resource-group myRG --cluster-name myAKS --name userpool --node-count 5

# Update autoscaler
az aks nodepool update \
  --resource-group myRG \
  --cluster-name myAKS \
  --name userpool \
  --update-cluster-autoscaler \
  --min-count 2 \
  --max-count 20

# Delete node pool
az aks nodepool delete --resource-group myRG --cluster-name myAKS --name userpool

# Add spot node pool
az aks nodepool add \
  --resource-group myRG \
  --cluster-name myAKS \
  --name spotnodes \
  --priority Spot \
  --eviction-policy Delete \
  --spot-max-price -1 \
  --node-count 3
```

### Cluster Start/Stop

```bash
# Stop cluster (save costs in dev/test)
az aks stop --resource-group myRG --name myAKS

# Start cluster
az aks start --resource-group myRG --name myAKS
```

---

# Chapter 16: Essential Azure Services for DevOps

## 16.1 Azure Container Registry (ACR)

```bash
# Create ACR
az acr create --resource-group myRG --name myacr --sku Premium --location eastus

# SKUs:
# Basic    : Dev/test, 10 GB storage
# Standard : Production, 100 GB storage
# Premium  : Geo-replication, private endpoints, 500 GB storage

# Login to ACR
az acr login --name myacr

# Build image using ACR Tasks
az acr build --registry myacr --image myapp:v1 .

# Push image
docker tag myapp:v1 myacr.azurecr.io/myapp:v1
docker push myacr.azurecr.io/myapp:v1

# List repositories
az acr repository list --name myacr --output table

# List tags
az acr repository show-tags --name myacr --repository myapp --output table

# Delete image
az acr repository delete --name myacr --image myapp:v1

# Enable admin user (for simple auth)
az acr update --name myacr --admin-enabled true
az acr credential show --name myacr

# Attach ACR to AKS (recommended)
az aks update --resource-group myRG --name myAKS --attach-acr myacr

# Or manually create pull secret
kubectl create secret docker-registry acr-secret \
  --docker-server=myacr.azurecr.io \
  --docker-username=<username> \
  --docker-password=<password>

# Enable vulnerability scanning
az acr config content-trust update --registry myacr --status enabled

# Geo-replication (Premium only)
az acr replication create --registry myacr --location westeurope

# Private endpoint
az acr private-endpoint-connection list --registry-name myacr

# ACR Tasks (automated builds)
az acr task create \
  --registry myacr \
  --name buildapp \
  --image myapp:{{.Run.ID}} \
  --context https://github.com/org/repo.git \
  --file Dockerfile \
  --git-access-token <token>
```

## 16.2 Azure Virtual Network (VNet)

```bash
# Create VNet
az network vnet create \
  --resource-group myRG \
  --name myVNet \
  --address-prefixes 10.0.0.0/8 \
  --location eastus

# Add subnet
az network vnet subnet create \
  --resource-group myRG \
  --vnet-name myVNet \
  --name aks-subnet \
  --address-prefixes 10.1.0.0/16

az network vnet subnet create \
  --resource-group myRG \
  --vnet-name myVNet \
  --name appgw-subnet \
  --address-prefixes 10.2.0.0/24

# List subnets
az network vnet subnet list --resource-group myRG --vnet-name myVNet --output table

# Create NSG
az network nsg create --resource-group myRG --name myNSG

# Add NSG rule
az network nsg rule create \
  --resource-group myRG \
  --nsg-name myNSG \
  --name AllowSSH \
  --priority 100 \
  --source-address-prefixes Internet \
  --destination-port-ranges 22 \
  --access Allow \
  --protocol Tcp

# Associate NSG with subnet
az network vnet subnet update \
  --resource-group myRG \
  --vnet-name myVNet \
  --name aks-subnet \
  --network-security-group myNSG

# VNet peering
az network vnet peering create \
  --resource-group myRG \
  --name peer-vnet1-to-vnet2 \
  --vnet-name myVNet \
  --remote-vnet /subscriptions/.../resourceGroups/otherRG/providers/Microsoft.Network/virtualNetworks/otherVNet \
  --allow-vnet-access
```

## 16.3 Azure Load Balancer and Application Gateway

```bash
# Standard Load Balancer (Layer 4)
# Usually created automatically by AKS for LoadBalancer services

# Application Gateway (Layer 7)
# Create public IP
az network public-ip create \
  --resource-group myRG \
  --name appgw-pip \
  --allocation-method Static \
  --sku Standard

# Create Application Gateway
az network application-gateway create \
  --resource-group myRG \
  --name myAppGateway \
  --location eastus \
  --sku Standard_v2 \
  --capacity 2 \
  --vnet-name myVNet \
  --subnet appgw-subnet \
  --public-ip-address appgw-pip \
  --priority 100

# Enable AGIC (Application Gateway Ingress Controller) on AKS
az aks enable-addons \
  --resource-group myRG \
  --name myAKS \
  --addons ingress-appgw \
  --appgw-id /subscriptions/.../resourceGroups/myRG/providers/Microsoft.Network/applicationGateways/myAppGateway
```

## 16.4 Azure Key Vault

```bash
# Create Key Vault
az keyvault create \
  --resource-group myRG \
  --name mykeyvault \
  --location eastus \
  --enable-rbac-authorization true

# Add secret
az keyvault secret set \
  --vault-name mykeyvault \
  --name "DatabasePassword" \
  --value "supersecret123"

# Get secret
az keyvault secret show --vault-name mykeyvault --name "DatabasePassword" --query "value"

# List secrets
az keyvault secret list --vault-name mykeyvault --output table

# Add certificate
az keyvault certificate import \
  --vault-name mykeyvault \
  --name "my-cert" \
  --file ./certificate.pfx \
  --password "certpassword"

# Create certificate from CA
az keyvault certificate create \
  --vault-name mykeyvault \
  --name "self-signed-cert" \
  --policy "$(az keyvault certificate get-default-policy)"

# Grant access (RBAC)
az role assignment create \
  --role "Key Vault Secrets User" \
  --assignee <principal-id> \
  --scope /subscriptions/.../resourceGroups/myRG/providers/Microsoft.KeyVault/vaults/mykeyvault

# Enable soft delete and purge protection
az keyvault update \
  --name mykeyvault \
  --enable-soft-delete true \
  --enable-purge-protection true

# Private endpoint
az keyvault private-endpoint-connection list --vault-name mykeyvault
```

## 16.5 Azure Storage

```bash
# Create storage account
az storage account create \
  --resource-group myRG \
  --name mystorageaccount \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2 \
  --access-tier Hot

# SKUs:
# Standard_LRS   : Locally redundant (3 copies in one datacenter)
# Standard_ZRS   : Zone redundant (3 copies across zones)
# Standard_GRS   : Geo-redundant (6 copies, 2 regions)
# Standard_RAGRS : Read-access geo-redundant
# Premium_LRS    : Premium SSD, locally redundant

# Create blob container
az storage container create \
  --account-name mystorageaccount \
  --name mycontainer

# Upload blob
az storage blob upload \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myfile.txt \
  --file ./myfile.txt

# Download blob
az storage blob download \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myfile.txt \
  --file ./downloaded.txt

# Create file share
az storage share create \
  --account-name mystorageaccount \
  --name myfileshare \
  --quota 100

# Get connection string
az storage account show-connection-string --name mystorageaccount --resource-group myRG

# Get storage key
az storage account keys list --account-name mystorageaccount --resource-group myRG

# Generate SAS token
az storage account generate-sas \
  --account-name mystorageaccount \
  --services b \
  --resource-types sco \
  --permissions rwlac \
  --expiry 2024-12-31
```

## 16.6 Azure Database Services

### Azure Database for PostgreSQL

```bash
# Create PostgreSQL Flexible Server
az postgres flexible-server create \
  --resource-group myRG \
  --name mypostgres \
  --location eastus \
  --admin-user adminuser \
  --admin-password "SuperSecure123!" \
  --sku-name Standard_B1ms \
  --tier Burstable \
  --storage-size 32 \
  --version 14

# Configure firewall
az postgres flexible-server firewall-rule create \
  --resource-group myRG \
  --name mypostgres \
  --rule-name AllowMyIP \
  --start-ip-address <my-ip> \
  --end-ip-address <my-ip>

# Allow Azure services
az postgres flexible-server firewall-rule create \
  --resource-group myRG \
  --name mypostgres \
  --rule-name AllowAzureServices \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 0.0.0.0

# Create database
az postgres flexible-server db create \
  --resource-group myRG \
  --server-name mypostgres \
  --database-name mydb

# Connection string
# postgresql://adminuser:SuperSecure123!@mypostgres.postgres.database.azure.com:5432/mydb
```

### Azure Cache for Redis

```bash
# Create Redis Cache
az redis create \
  --resource-group myRG \
  --name myredis \
  --location eastus \
  --sku Basic \
  --vm-size C0

# SKUs:
# Basic    : Single node, dev/test
# Standard : Primary/replica, SLA
# Premium  : Clustering, persistence, VNet

# Get connection info
az redis list-keys --resource-group myRG --name myredis
az redis show --resource-group myRG --name myredis --query "hostName"
```

## 16.7 Azure Monitor and Log Analytics

```bash
# Create Log Analytics workspace
az monitor log-analytics workspace create \
  --resource-group myRG \
  --workspace-name myworkspace \
  --location eastus

# Get workspace ID
az monitor log-analytics workspace show \
  --resource-group myRG \
  --workspace-name myworkspace \
  --query "customerId"

# Get workspace key
az monitor log-analytics workspace get-shared-keys \
  --resource-group myRG \
  --workspace-name myworkspace

# Query logs (KQL)
az monitor log-analytics query \
  --workspace <workspace-id> \
  --analytics-query "ContainerLogV2 | where TimeGenerated > ago(1h) | take 10"

# Create alert rule
az monitor metrics alert create \
  --resource-group myRG \
  --name "High CPU Alert" \
  --scopes /subscriptions/.../resourceGroups/myRG/providers/Microsoft.ContainerService/managedClusters/myAKS \
  --condition "avg node_cpu_usage_percentage > 80" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action-group /subscriptions/.../resourceGroups/myRG/providers/Microsoft.Insights/actionGroups/myActionGroup
```

## 16.8 Azure Private DNS

```bash
# Create private DNS zone
az network private-dns zone create \
  --resource-group myRG \
  --name "privatelink.azurecr.io"

# Link to VNet
az network private-dns link vnet create \
  --resource-group myRG \
  --zone-name "privatelink.azurecr.io" \
  --name "mylink" \
  --virtual-network myVNet \
  --registration-enabled false

# Add A record
az network private-dns record-set a add-record \
  --resource-group myRG \
  --zone-name "privatelink.azurecr.io" \
  --record-set-name myacr \
  --ipv4-address 10.0.0.5

# Common private DNS zones for Azure services:
# privatelink.azurecr.io             - ACR
# privatelink.blob.core.windows.net  - Blob Storage
# privatelink.file.core.windows.net  - Azure Files
# privatelink.vaultcore.azure.net    - Key Vault
# privatelink.database.windows.net   - SQL Database
# privatelink.postgres.database.azure.com - PostgreSQL
```

---

# Chapter 17: Azure CLI Essentials

## 17.1 Azure CLI Basics

```bash
# Login
az login                              # Browser-based
az login --use-device-code            # Device code flow
az login --service-principal -u <app-id> -p <password> --tenant <tenant-id>

# Set subscription
az account list --output table
az account set --subscription "My Subscription"
az account show

# Common output formats
az <command> --output table           # Table format
az <command> --output json            # JSON format
az <command> --output yaml            # YAML format
az <command> --output tsv             # Tab-separated (for scripts)

# Query with JMESPath
az vm list --query "[].{name:name, state:powerState}" --output table
az vm list --query "[?powerState=='VM running'].name"
az aks show -n myAKS -g myRG --query "nodeResourceGroup"

# Resource groups
az group create --name myRG --location eastus
az group list --output table
az group delete --name myRG --yes

# Resource management
az resource list --resource-group myRG --output table
az resource show --ids /subscriptions/.../resourceGroups/myRG/providers/...
az resource delete --ids /subscriptions/.../resourceGroups/myRG/providers/...

# Tags
az resource tag --tags Environment=Production Team=DevOps --ids <resource-id>
az resource list --tag Environment=Production

# Locks
az lock create --name CanNotDelete --resource-group myRG --lock-type CanNotDelete
az lock list --resource-group myRG
```

## 17.2 Common Azure CLI Patterns

```bash
# Get resource ID
RESOURCE_ID=$(az <resource> show -n <name> -g <rg> --query id -o tsv)

# Wait for operation
az <resource> wait --created --ids $RESOURCE_ID
az <resource> wait --deleted --ids $RESOURCE_ID
az <resource> wait --updated --ids $RESOURCE_ID

# Export ARM template
az group export --name myRG > template.json

# Deploy ARM template
az deployment group create \
  --resource-group myRG \
  --template-file template.json \
  --parameters @parameters.json

# What-if deployment
az deployment group what-if \
  --resource-group myRG \
  --template-file template.json

# Find available VM sizes
az vm list-sizes --location eastus --output table

# Find available Kubernetes versions
az aks get-versions --location eastus --output table

# Configure defaults
az config set defaults.location=eastus defaults.group=myRG
```

---

This concludes Part 5 covering AKS and Azure Services.

**Continue to Part 6 for:**
- Jenkins CI/CD Complete Guide
- Pipeline Development
- Integration with Kubernetes
