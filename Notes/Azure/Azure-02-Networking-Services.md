# Azure Networking Services - Complete Guide
## VNet, NSG, Load Balancers, Firewalls, VPN, Private Link

---

# 1. VIRTUAL NETWORK (VNet)

## What It Is
Azure Virtual Network is the fundamental building block for your private network in Azure. It enables Azure resources to securely communicate with each other, the internet, and on-premises networks.

## Key Concepts

### Address Space
```
CIDR NOTATION:
10.0.0.0/16 = 65,536 IP addresses
10.0.0.0/24 = 256 IP addresses
10.0.0.0/27 = 32 IP addresses

RESERVED IPs IN EACH SUBNET (5 addresses):
- x.x.x.0: Network address
- x.x.x.1: Default gateway
- x.x.x.2, x.x.x.3: Azure DNS
- x.x.x.255: Broadcast

Example: 10.0.1.0/24 has 251 usable IPs (256 - 5)
```

### VNet Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│  VNet: 10.0.0.0/16                                              │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Subnet: Web-Tier (10.0.1.0/24)                              ││
│  │ VMs, App Gateway                                            ││
│  └─────────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Subnet: App-Tier (10.0.2.0/24)                              ││
│  │ VMs, AKS                                                    ││
│  └─────────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Subnet: DB-Tier (10.0.3.0/24)                               ││
│  │ Azure SQL, Private Endpoints                                ││
│  └─────────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Subnet: AzureBastionSubnet (10.0.4.0/27)                    ││
│  │ Azure Bastion                                               ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

### VNet Peering
```
VNET PEERING:
- Connect VNets (same or different regions)
- Traffic stays on Microsoft backbone
- Low latency, high bandwidth
- Non-transitive (A↔B, B↔C doesn't mean A↔C)
- Use Gateway Transit for hub-spoke

GLOBAL VNET PEERING:
- VNets in different regions
- Same benefits as regional peering
- Bandwidth charges apply
```

## CLI Commands

```bash
# Create VNet
az network vnet create \
  --resource-group myRG \
  --name myVNet \
  --address-prefix 10.0.0.0/16 \
  --subnet-name WebSubnet \
  --subnet-prefix 10.0.1.0/24

# Add subnet
az network vnet subnet create \
  --resource-group myRG \
  --vnet-name myVNet \
  --name AppSubnet \
  --address-prefix 10.0.2.0/24

# List VNets
az network vnet list -o table

# Create VNet peering
az network vnet peering create \
  --resource-group myRG \
  --name VNet1-to-VNet2 \
  --vnet-name VNet1 \
  --remote-vnet VNet2 \
  --allow-vnet-access

# Show VNet details
az network vnet show -g myRG -n myVNet
```

---

# 2. NETWORK SECURITY GROUPS (NSG)

## What It Is
NSG contains security rules that allow or deny inbound/outbound network traffic. Can be associated with subnets or NICs.

## Key Concepts

### Rule Processing
```
RULE PRIORITY:
- Lower number = higher priority
- Rules: 100-4096 (custom)
- Default rules: 65000-65500

PROCESSING ORDER:
1. NSG on subnet (if exists)
2. NSG on NIC (if exists)
3. Most restrictive wins for deny
4. Allow needs to pass both

DEFAULT RULES (Cannot delete):
┌────────────────────────────────────────────────────────────────┐
│ Inbound:                                                       │
│ - AllowVNetInBound (65000): Allow VNet traffic                │
│ - AllowAzureLoadBalancerInBound (65001): Allow LB probes      │
│ - DenyAllInBound (65500): Deny everything else                │
│                                                                │
│ Outbound:                                                      │
│ - AllowVNetOutBound (65000): Allow VNet traffic               │
│ - AllowInternetOutBound (65001): Allow internet               │
│ - DenyAllOutBound (65500): Deny everything else               │
└────────────────────────────────────────────────────────────────┘
```

### Service Tags
```
COMMON SERVICE TAGS:
- VirtualNetwork: All VNet addresses
- AzureLoadBalancer: Azure LB probes
- Internet: Public IP addresses
- AzureCloud: All Azure datacenter IPs
- Storage: Azure Storage IPs
- Sql: Azure SQL IPs
- AzureKeyVault: Key Vault IPs
- AzureActiveDirectory: Azure AD IPs
```

## CLI Commands

```bash
# Create NSG
az network nsg create \
  --resource-group myRG \
  --name myNSG

# Add inbound rule (allow HTTP)
az network nsg rule create \
  --resource-group myRG \
  --nsg-name myNSG \
  --name AllowHTTP \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --destination-port-ranges 80 443 \
  --source-address-prefixes Internet \
  --destination-address-prefixes '*'

# Add outbound rule (deny internet except specific)
az network nsg rule create \
  --resource-group myRG \
  --nsg-name myNSG \
  --name DenyInternet \
  --priority 200 \
  --direction Outbound \
  --access Deny \
  --protocol '*' \
  --destination-port-ranges '*' \
  --source-address-prefixes '*' \
  --destination-address-prefixes Internet

# Associate NSG with subnet
az network vnet subnet update \
  --resource-group myRG \
  --vnet-name myVNet \
  --name WebSubnet \
  --network-security-group myNSG

# List NSG rules
az network nsg rule list -g myRG --nsg-name myNSG -o table
```

---

# 3. APPLICATION SECURITY GROUPS (ASG)

## What It Is
ASGs enable you to group VMs logically and define NSG rules based on those groups rather than IP addresses.

## When to Use
- Dynamic environments where IPs change
- Microservices with multiple tiers
- Simplify NSG rule management

## Example
```
WITHOUT ASG:
Rule: Allow 10.0.1.4, 10.0.1.5, 10.0.1.6 → 10.0.2.4, 10.0.2.5

WITH ASG:
Rule: Allow WebServers-ASG → AppServers-ASG
(VMs can join/leave ASGs dynamically)
```

## CLI Commands

```bash
# Create ASGs
az network asg create -g myRG -n WebServers
az network asg create -g myRG -n AppServers

# Associate NIC with ASG
az network nic ip-config update \
  --resource-group myRG \
  --nic-name myNIC \
  --name ipconfig1 \
  --application-security-groups WebServers

# Create NSG rule using ASGs
az network nsg rule create \
  --resource-group myRG \
  --nsg-name myNSG \
  --name AllowWebToApp \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --destination-port-ranges 8080 \
  --source-asgs WebServers \
  --destination-asgs AppServers
```

---

# 4. AZURE LOAD BALANCER

## What It Is
Layer 4 (TCP/UDP) load balancer. Distributes traffic across VMs in a backend pool.

## Types
```
PUBLIC LOAD BALANCER:
- Has public IP
- Distributes internet traffic to VMs
- Outbound NAT for VMs

INTERNAL LOAD BALANCER:
- Private IP only
- Distributes traffic within VNet
- Used for internal services
```

## SKUs
```
BASIC (Legacy):
- Free
- Up to 300 instances
- No SLA
- No Availability Zones

STANDARD:
- Production workloads
- Up to 1000 instances
- 99.99% SLA
- Availability Zones support
- More features
```

## CLI Commands

```bash
# Create public IP for LB
az network public-ip create \
  --resource-group myRG \
  --name myLBPublicIP \
  --sku Standard \
  --zone 1 2 3

# Create load balancer
az network lb create \
  --resource-group myRG \
  --name myLB \
  --sku Standard \
  --public-ip-address myLBPublicIP \
  --frontend-ip-name myFrontEnd \
  --backend-pool-name myBackEndPool

# Create health probe
az network lb probe create \
  --resource-group myRG \
  --lb-name myLB \
  --name myHealthProbe \
  --protocol tcp \
  --port 80

# Create load balancing rule
az network lb rule create \
  --resource-group myRG \
  --lb-name myLB \
  --name myLBRule \
  --protocol tcp \
  --frontend-port 80 \
  --backend-port 80 \
  --frontend-ip-name myFrontEnd \
  --backend-pool-name myBackEndPool \
  --probe-name myHealthProbe

# Add VM to backend pool
az network nic ip-config address-pool add \
  --resource-group myRG \
  --nic-name myNIC \
  --ip-config-name ipconfig1 \
  --lb-name myLB \
  --address-pool myBackEndPool
```

---

# 5. APPLICATION GATEWAY

## What It Is
Layer 7 (HTTP/HTTPS) load balancer with Web Application Firewall (WAF), SSL termination, URL-based routing, and more.

## Key Features
```
FEATURES:
┌─────────────────────────────────────────────────────────────────┐
│ - SSL/TLS termination                                          │
│ - URL-based routing (/images/* → pool1, /api/* → pool2)       │
│ - Multi-site hosting (host headers)                            │
│ - Web Application Firewall (WAF)                               │
│ - Session affinity (cookie-based)                              │
│ - WebSocket support                                            │
│ - Autoscaling                                                  │
│ - Zone redundancy                                              │
│ - Private Link support                                         │
└─────────────────────────────────────────────────────────────────┘
```

## Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│  APPLICATION GATEWAY                                            │
│                                                                 │
│  Frontend IP ──► Listener ──► Rules ──► Backend Pool           │
│  (Public/Private) (HTTP/HTTPS)  (Routing)  (VMs/VMSS/AKS)      │
│                                                                 │
│  Components:                                                    │
│  - Frontend IP: Public and/or Private                          │
│  - Listeners: HTTP (80), HTTPS (443)                           │
│  - Rules: Basic or Path-based                                  │
│  - HTTP Settings: Protocol, port, timeout                      │
│  - Backend Pool: Targets (VMs, IPs, FQDN)                      │
│  - Health Probes: Check backend health                         │
└─────────────────────────────────────────────────────────────────┘
```

## CLI Commands

```bash
# Create Application Gateway
az network application-gateway create \
  --resource-group myRG \
  --name myAppGW \
  --location eastus \
  --sku Standard_v2 \
  --capacity 2 \
  --vnet-name myVNet \
  --subnet AppGWSubnet \
  --public-ip-address myAppGWPublicIP \
  --frontend-port 80 \
  --http-settings-port 80 \
  --http-settings-protocol Http \
  --routing-rule-type Basic

# Add backend pool
az network application-gateway address-pool create \
  --resource-group myRG \
  --gateway-name myAppGW \
  --name myBackendPool \
  --servers 10.0.1.4 10.0.1.5

# Add path-based rule
az network application-gateway url-path-map create \
  --resource-group myRG \
  --gateway-name myAppGW \
  --name myPathMap \
  --paths "/api/*" \
  --address-pool apiPool \
  --http-settings myHttpSettings
```

---

# 6. AZURE FRONT DOOR

## What It Is
Global, scalable entry-point that uses Microsoft's global edge network. Provides global load balancing, SSL offload, WAF, and CDN capabilities.

## When to Use
- Global applications
- Need CDN + WAF + Load Balancing
- Latency-sensitive applications
- Multi-region deployment

## Key Features
```
CAPABILITIES:
- Global HTTP load balancing
- Instant failover
- URL-based routing
- Session affinity
- SSL termination at edge
- Web Application Firewall (WAF)
- Caching (CDN)
- URL rewrite/redirect
- Private Link to backends
```

## CLI Commands

```bash
# Create Front Door
az network front-door create \
  --resource-group myRG \
  --name myFrontDoor \
  --backend-address myapp.azurewebsites.net

# Add routing rule
az network front-door routing-rule create \
  --resource-group myRG \
  --front-door-name myFrontDoor \
  --name myRoutingRule \
  --frontend-endpoints DefaultFrontendEndpoint \
  --backend-pool DefaultBackendPool \
  --route-type Forward
```

---

# 7. AZURE FIREWALL

## What It Is
Managed, cloud-based network security service. Provides stateful firewall with built-in high availability and unrestricted cloud scalability.

## Key Features
```
FEATURES:
┌─────────────────────────────────────────────────────────────────┐
│ - Application FQDN filtering                                   │
│ - Network traffic filtering                                    │
│ - Threat intelligence                                          │
│ - NAT rules (DNAT/SNAT)                                       │
│ - Built-in high availability                                   │
│ - Availability Zones support                                   │
│ - Forced tunneling                                             │
│ - DNS proxy                                                    │
│ - TLS inspection (Premium)                                     │
│ - Web categories (Premium)                                     │
└─────────────────────────────────────────────────────────────────┘
```

## SKUs
```
STANDARD:
- Network rules (IP, port, protocol)
- Application rules (FQDN)
- NAT rules
- Threat intelligence

PREMIUM:
- All Standard features
- TLS inspection
- IDPS (Intrusion Detection)
- URL filtering
- Web categories
```

## CLI Commands

```bash
# Create Azure Firewall
az network firewall create \
  --resource-group myRG \
  --name myFirewall \
  --location eastus \
  --vnet-name myVNet \
  --sku AZFW_VNet \
  --tier Standard

# Create firewall policy
az network firewall policy create \
  --resource-group myRG \
  --name myFirewallPolicy \
  --tier Standard

# Add network rule
az network firewall policy rule-collection-group create \
  --resource-group myRG \
  --policy-name myFirewallPolicy \
  --name myRuleCollectionGroup \
  --priority 100

az network firewall policy rule-collection-group collection add-filter-collection \
  --resource-group myRG \
  --policy-name myFirewallPolicy \
  --rcg-name myRuleCollectionGroup \
  --name AllowWeb \
  --collection-priority 100 \
  --rule-type NetworkRule \
  --action Allow \
  --rule-name AllowHTTP \
  --source-addresses 10.0.0.0/24 \
  --destination-addresses '*' \
  --destination-ports 80 443 \
  --ip-protocols TCP
```

---

# 8. PRIVATE LINK & PRIVATE ENDPOINTS

## What It Is
Private Link enables you to access Azure PaaS services (Storage, SQL, etc.) over a private endpoint in your VNet. Traffic stays on Microsoft backbone.

## Key Concepts
```
PRIVATE ENDPOINT:
- NIC with private IP in your VNet
- Connects to specific Azure service instance
- Traffic never goes to internet

PRIVATE LINK SERVICE:
- Expose your own service via Private Link
- Consumers connect via Private Endpoint

BENEFITS:
- No public IP needed
- Traffic on Microsoft backbone
- Protection against data exfiltration
- Works across regions and tenants
```

## CLI Commands

```bash
# Create private endpoint for storage
az network private-endpoint create \
  --resource-group myRG \
  --name myStoragePE \
  --vnet-name myVNet \
  --subnet PrivateEndpointSubnet \
  --private-connection-resource-id /subscriptions/.../storageAccounts/mystorageaccount \
  --group-id blob \
  --connection-name myStorageConnection

# Create private DNS zone
az network private-dns zone create \
  --resource-group myRG \
  --name privatelink.blob.core.windows.net

# Link DNS zone to VNet
az network private-dns link vnet create \
  --resource-group myRG \
  --zone-name privatelink.blob.core.windows.net \
  --name myDNSLink \
  --virtual-network myVNet \
  --registration-enabled false

# Create DNS record for private endpoint
az network private-endpoint dns-zone-group create \
  --resource-group myRG \
  --endpoint-name myStoragePE \
  --name myZoneGroup \
  --private-dns-zone privatelink.blob.core.windows.net \
  --zone-name blob
```

---

# 9. VPN GATEWAY

## What It Is
Connects on-premises networks to Azure VNets over encrypted tunnels (IPsec/IKE).

## Types
```
SITE-TO-SITE (S2S):
- On-premises to Azure
- Requires VPN device on-premises
- IPsec/IKE tunnel

POINT-TO-SITE (P2S):
- Individual computer to Azure
- No VPN device needed
- OpenVPN, IKEv2, SSTP

VNET-TO-VNET:
- Azure VNet to Azure VNet
- Encrypted over internet
- Alternative to VNet peering
```

## SKUs
```
┌──────────────────┬─────────────┬───────────────┬──────────────┐
│ SKU              │ Tunnels     │ Throughput    │ Zone Support │
├──────────────────┼─────────────┼───────────────┼──────────────┤
│ Basic            │ 10          │ 100 Mbps      │ No           │
│ VpnGw1           │ 30          │ 650 Mbps      │ Optional     │
│ VpnGw2           │ 30          │ 1 Gbps        │ Optional     │
│ VpnGw3           │ 30          │ 1.25 Gbps     │ Optional     │
│ VpnGw4           │ 100         │ 5 Gbps        │ Optional     │
│ VpnGw5           │ 100         │ 10 Gbps       │ Optional     │
└──────────────────┴─────────────┴───────────────┴──────────────┘
```

---

# 10. EXPRESSROUTE

## What It Is
Private, dedicated connection from on-premises to Azure. Doesn't go over public internet. Provided by connectivity partners.

## Key Features
```
BENEFITS:
- Private connection (not internet)
- Higher bandwidth (50 Mbps to 100 Gbps)
- Lower latency
- More reliable
- SLA up to 99.95%

CONNECTIVITY MODELS:
- CloudExchange co-location
- Point-to-point Ethernet
- Any-to-any (IPVPN)
- ExpressRoute Direct
```

---

# 11. AZURE BASTION

## What It Is
Fully managed PaaS service for secure RDP/SSH access to VMs without exposing public IPs.

## Benefits
```
- No public IPs on VMs
- No NSG rules for RDP/SSH
- Protection against port scanning
- Hardened against zero-day exploits
- Works through Azure Portal
- No agent needed on VM
```

## CLI Commands

```bash
# Create Bastion
az network bastion create \
  --resource-group myRG \
  --name myBastion \
  --public-ip-address myBastionIP \
  --vnet-name myVNet \
  --location eastus
```

---

# 12. AZURE DNS

## What It Is
Hosting service for DNS domains using Azure infrastructure.

## Key Features
```
PUBLIC DNS ZONES:
- Host your domain (example.com)
- Resolve to public IPs
- Supports all record types

PRIVATE DNS ZONES:
- Name resolution within VNets
- No custom DNS server needed
- Auto-registration of VM names
```

## CLI Commands

```bash
# Create public DNS zone
az network dns zone create \
  --resource-group myRG \
  --name example.com

# Add A record
az network dns record-set a add-record \
  --resource-group myRG \
  --zone-name example.com \
  --record-set-name www \
  --ipv4-address 1.2.3.4

# Create private DNS zone
az network private-dns zone create \
  --resource-group myRG \
  --name private.example.com

# Link to VNet
az network private-dns link vnet create \
  --resource-group myRG \
  --zone-name private.example.com \
  --name myLink \
  --virtual-network myVNet \
  --registration-enabled true  # Auto-register VM names
```

---

# QUICK REFERENCE: NETWORKING COMPARISON

| Service | Layer | Use Case | Key Feature |
|---------|-------|----------|-------------|
| NSG | 4 | Subnet/NIC firewall | Stateful rules |
| Azure Firewall | 3-7 | Centralized firewall | FQDN filtering |
| Load Balancer | 4 | TCP/UDP distribution | Health probes |
| Application Gateway | 7 | HTTP load balancing | WAF, URL routing |
| Front Door | 7 | Global load balancing | Edge CDN + WAF |
| VPN Gateway | 3 | Site-to-site VPN | Encrypted tunnel |
| ExpressRoute | 2 | Private connection | Dedicated link |
| Private Link | - | Private PaaS access | No public IP |
| Bastion | - | Secure VM access | No public VM IP |
