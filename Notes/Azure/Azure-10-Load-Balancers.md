# Azure Load Balancers - Complete Deep Dive
## Load Balancer, Application Gateway, Front Door, Traffic Manager

---

# UNDERSTANDING LOAD BALANCING

## Why Load Balancing?

```
WITHOUT LOAD BALANCER:
┌─────────┐
│  Users  │────────────────────►  Single Server  ────► SINGLE POINT OF FAILURE
└─────────┘

WITH LOAD BALANCER:
┌─────────┐      ┌──────────────┐      ┌─────────────┐
│  Users  │─────►│ Load Balancer│─────►│  Server 1   │
└─────────┘      └──────────────┘      ├─────────────┤
                        │              │  Server 2   │
                        │              ├─────────────┤
                        └─────────────►│  Server 3   │
                                       └─────────────┘
```

## Benefits
- **High Availability**: No single point of failure
- **Scalability**: Add/remove servers dynamically
- **Performance**: Distribute load evenly
- **Health Monitoring**: Auto-remove unhealthy servers
- **Maintenance**: Update servers without downtime

---

# OSI MODEL & LOAD BALANCING LAYERS

```
OSI LAYER MODEL:
┌─────────────────────────────────────────────────────────────────┐
│ Layer 7 - Application  │ HTTP, HTTPS, WebSocket                │
│                        │ URL path, headers, cookies            │
│                        │ ► Application Gateway, Front Door     │
├────────────────────────┼────────────────────────────────────────┤
│ Layer 4 - Transport    │ TCP, UDP                              │
│                        │ IP address, port number               │
│                        │ ► Azure Load Balancer                 │
├────────────────────────┼────────────────────────────────────────┤
│ Layer 3 - Network      │ IP routing                            │
│                        │ DNS-based routing                     │
│                        │ ► Traffic Manager                     │
└─────────────────────────────────────────────────────────────────┘
```

---

# 1. AZURE LOAD BALANCER (Layer 4)

## What It Is
Layer 4 (TCP/UDP) load balancer that distributes traffic based on IP address and port. Does NOT inspect packet contents - fastest and most efficient.

## How It Works
```
PACKET FLOW:
┌────────────┐     ┌─────────────────┐     ┌────────────┐
│   Client   │────►│  Load Balancer  │────►│  Backend   │
│ 1.2.3.4    │     │  Frontend IP    │     │  VM Pool   │
└────────────┘     │  20.30.40.50    │     └────────────┘
                   └─────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
   ┌─────────┐       ┌─────────┐       ┌─────────┐
   │  VM 1   │       │  VM 2   │       │  VM 3   │
   │ :8080   │       │ :8080   │       │ :8080   │
   └─────────┘       └─────────┘       └─────────┘
```

## Types

### Public Load Balancer
```
PURPOSE: Internet-facing traffic distribution

┌──────────────┐
│   Internet   │
└──────┬───────┘
       │
       ▼
┌──────────────┐     Public IP: 20.30.40.50
│  Public LB   │     
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  Backend VMs │     Private IPs: 10.0.0.x
└──────────────┘

USE CASES:
- Web applications
- API endpoints
- Any internet-facing service
```

### Internal Load Balancer
```
PURPOSE: Internal traffic within VNet

┌──────────────┐
│  Web Tier    │     10.0.1.x
└──────┬───────┘
       │
       ▼
┌──────────────┐     Private IP: 10.0.2.100
│ Internal LB  │     
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  App Tier    │     10.0.2.x
└──────────────┘

USE CASES:
- Multi-tier applications
- Database load balancing
- Internal microservices
```

## SKUs (Stock Keeping Units)

```
┌─────────────────────────────────────────────────────────────────┐
│ Feature              │ Basic              │ Standard            │
├──────────────────────┼────────────────────┼─────────────────────┤
│ Backend pool size    │ Up to 300          │ Up to 1000          │
│ Health probes        │ TCP, HTTP          │ TCP, HTTP, HTTPS    │
│ Availability Zones   │ ❌                 │ ✅ Zone-redundant   │
│ SLA                  │ None               │ 99.99%              │
│ Security             │ Open by default    │ Closed by default   │
│ Outbound rules       │ ❌                 │ ✅                  │
│ Multiple frontends   │ ❌                 │ ✅                  │
│ HA Ports             │ ❌                 │ ✅                  │
│ Cost                 │ Free               │ Pay per rule/data   │
└─────────────────────────────────────────────────────────────────┘

RECOMMENDATION: Always use Standard for production
```

## Components Deep Dive

### Frontend IP Configuration
```
WHAT: The IP address clients connect to

PUBLIC FRONTEND:
- Uses Azure Public IP
- Internet-accessible
- Can have multiple frontends

PRIVATE FRONTEND:
- Uses VNet IP address
- Internal access only
- For internal load balancing

az network lb frontend-ip create \
  --resource-group myRG \
  --lb-name myLB \
  --name myFrontend \
  --public-ip-address myPublicIP
```

### Backend Pool
```
WHAT: Collection of VMs/instances that receive traffic

BACKEND TYPES:
┌────────────────────────────────────────────────────────────────┐
│ NIC-based:     Add individual VM NICs                         │
│ IP-based:      Add by IP address (Standard SKU only)          │
│ VMSS:          Virtual Machine Scale Set                      │
└────────────────────────────────────────────────────────────────┘

# Add VM to backend pool
az network nic ip-config address-pool add \
  --resource-group myRG \
  --nic-name myNIC \
  --ip-config-name ipconfig1 \
  --lb-name myLB \
  --address-pool myBackendPool
```

### Health Probes
```
WHAT: Checks if backend instances are healthy

PROBE TYPES:
┌────────────────────────────────────────────────────────────────┐
│ TCP Probe:                                                     │
│ - Checks if port is open                                      │
│ - Simple connectivity check                                    │
│ - Use for: Non-HTTP services                                  │
│                                                                │
│ HTTP/HTTPS Probe:                                              │
│ - Sends HTTP request                                          │
│ - Expects 200 OK response                                     │
│ - Can check specific path                                     │
│ - Use for: Web applications                                   │
└────────────────────────────────────────────────────────────────┘

PROBE SETTINGS:
- Port: Which port to check
- Interval: How often (default 15 sec)
- Unhealthy threshold: Failures before marking unhealthy (default 2)

# Create HTTP health probe
az network lb probe create \
  --resource-group myRG \
  --lb-name myLB \
  --name myHealthProbe \
  --protocol Http \
  --port 80 \
  --path /health \
  --interval 15 \
  --threshold 2
```

### Load Balancing Rules
```
WHAT: Defines how traffic is distributed

RULE COMPONENTS:
┌────────────────────────────────────────────────────────────────┐
│ Frontend IP + Port  ──►  Backend Pool + Port                  │
│                                                                │
│ Example:                                                       │
│ 20.30.40.50:80  ──►  BackendPool:8080                         │
└────────────────────────────────────────────────────────────────┘

DISTRIBUTION MODES:
┌────────────────────────────────────────────────────────────────┐
│ 5-tuple hash (default):                                        │
│ - Source IP, Source Port, Dest IP, Dest Port, Protocol        │
│ - Best distribution                                            │
│                                                                │
│ Source IP affinity (2-tuple):                                  │
│ - Source IP, Dest IP                                          │
│ - Same client → Same server                                   │
│ - Use for: Session persistence                                │
│                                                                │
│ Source IP + Protocol (3-tuple):                                │
│ - Source IP, Dest IP, Protocol                                │
│ - Session persistence per protocol                            │
└────────────────────────────────────────────────────────────────┘

# Create load balancing rule
az network lb rule create \
  --resource-group myRG \
  --lb-name myLB \
  --name myLBRule \
  --protocol Tcp \
  --frontend-port 80 \
  --backend-port 8080 \
  --frontend-ip-name myFrontend \
  --backend-pool-name myBackendPool \
  --probe-name myHealthProbe \
  --idle-timeout 4 \
  --enable-tcp-reset true
```

### Outbound Rules (Standard SKU)
```
WHAT: Controls outbound (egress) connectivity

WHY NEEDED:
- VMs in backend pool need outbound internet access
- Without explicit rule, uses SNAT on LB frontend IP
- Can run out of SNAT ports!

SNAT PORT EXHAUSTION:
┌────────────────────────────────────────────────────────────────┐
│ Each frontend IP = 64,000 SNAT ports                          │
│ Ports shared across all backend VMs                           │
│                                                                │
│ Example:                                                       │
│ 1 frontend IP, 100 VMs = 640 ports per VM                     │
│ If VM needs 1000 connections → SNAT EXHAUSTION!               │
│                                                                │
│ SOLUTION: Add more frontend IPs or use NAT Gateway            │
└────────────────────────────────────────────────────────────────┘

# Create outbound rule
az network lb outbound-rule create \
  --resource-group myRG \
  --lb-name myLB \
  --name myOutboundRule \
  --frontend-ip-configs myFrontend \
  --address-pool myBackendPool \
  --protocol All \
  --outbound-ports 10000
```

### HA Ports (Standard SKU)
```
WHAT: Load balance ALL ports with single rule

USE CASE: Network Virtual Appliances (NVAs)

# Instead of creating rules for each port:
az network lb rule create \
  --resource-group myRG \
  --lb-name myLB \
  --name HAPortsRule \
  --protocol All \
  --frontend-port 0 \
  --backend-port 0 \
  --frontend-ip-name myFrontend \
  --backend-pool-name myBackendPool
```

## Complete CLI Example

```bash
# 1. Create resource group
az group create --name myRG --location eastus

# 2. Create public IP
az network public-ip create \
  --resource-group myRG \
  --name myPublicIP \
  --sku Standard \
  --zone 1 2 3

# 3. Create load balancer
az network lb create \
  --resource-group myRG \
  --name myLoadBalancer \
  --sku Standard \
  --public-ip-address myPublicIP \
  --frontend-ip-name myFrontEnd \
  --backend-pool-name myBackEndPool

# 4. Create health probe
az network lb probe create \
  --resource-group myRG \
  --lb-name myLoadBalancer \
  --name myHealthProbe \
  --protocol tcp \
  --port 80

# 5. Create load balancing rule
az network lb rule create \
  --resource-group myRG \
  --lb-name myLoadBalancer \
  --name myHTTPRule \
  --protocol tcp \
  --frontend-port 80 \
  --backend-port 80 \
  --frontend-ip-name myFrontEnd \
  --backend-pool-name myBackEndPool \
  --probe-name myHealthProbe \
  --idle-timeout 4 \
  --enable-tcp-reset true

# 6. Create VMs and add to backend pool
# (Create VMs with NICs in same VNet)

# 7. Add NIC to backend pool
az network nic ip-config address-pool add \
  --address-pool myBackEndPool \
  --ip-config-name ipconfig1 \
  --nic-name myNIC1 \
  --resource-group myRG \
  --lb-name myLoadBalancer
```

---

# 2. APPLICATION GATEWAY (Layer 7)

## What It Is
Layer 7 (HTTP/HTTPS) load balancer with advanced features: SSL termination, URL-based routing, Web Application Firewall (WAF), cookie-based session affinity.

## Key Difference from Load Balancer
```
AZURE LOAD BALANCER (L4):
- Sees: IP + Port
- Routes: Based on connection tuple
- Fast, simple, any protocol

APPLICATION GATEWAY (L7):
- Sees: Full HTTP request (URL, headers, cookies)
- Routes: Based on URL path, host header, etc.
- Feature-rich, HTTP/HTTPS only
```

## Architecture
```
APPLICATION GATEWAY COMPONENTS:
┌─────────────────────────────────────────────────────────────────┐
│                      APPLICATION GATEWAY                        │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────┐ │
│  │  Frontend   │───►│   Rules     │───►│   Backend Pools     │ │
│  │  (Listener) │    │  (Routing)  │    │   (Targets)         │ │
│  └─────────────┘    └─────────────┘    └─────────────────────┘ │
│                                                                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────┐ │
│  │   HTTP      │    │ Path-based  │    │  Pool 1: VMs        │ │
│  │  Settings   │    │ Host-based  │    │  Pool 2: VMSS       │ │
│  │             │    │ Redirect    │    │  Pool 3: App Service│ │
│  └─────────────┘    └─────────────┘    └─────────────────────┘ │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                    WAF (Optional)                           ││
│  │    OWASP Rules, Custom Rules, Bot Protection               ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

## SKUs

```
APPLICATION GATEWAY SKUS:
┌─────────────────────────────────────────────────────────────────┐
│ Standard v2:                                                    │
│ - Autoscaling                                                  │
│ - Zone redundancy                                              │
│ - Static VIP                                                   │
│ - Header rewrite                                               │
│ - AKS Ingress Controller                                       │
│                                                                 │
│ WAF v2:                                                         │
│ - All Standard v2 features                                     │
│ - Web Application Firewall                                     │
│ - OWASP 3.x rules                                              │
│ - Bot protection                                               │
│ - Custom rules                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Key Features

### URL Path-Based Routing
```
SCENARIO: Route based on URL path

https://myapp.com/images/*  ──►  Images Backend Pool (CDN/Storage)
https://myapp.com/api/*     ──►  API Backend Pool (App servers)
https://myapp.com/*         ──►  Default Backend Pool (Web servers)

# Path-based rule routes different paths to different backends
```

### Host-Based Routing (Multi-site)
```
SCENARIO: Multiple domains on same App Gateway

app1.contoso.com  ──►  App1 Backend Pool
app2.contoso.com  ──►  App2 Backend Pool
api.contoso.com   ──►  API Backend Pool

# Single App Gateway serves multiple applications
```

### SSL/TLS Termination
```
SSL TERMINATION:
┌─────────┐      HTTPS       ┌─────────────┐      HTTP      ┌─────────┐
│ Client  │─────────────────►│ App Gateway │───────────────►│ Backend │
└─────────┘    (Encrypted)   └─────────────┘  (Unencrypted) └─────────┘

BENEFITS:
- Offload SSL processing from backend
- Centralized certificate management
- Inspect traffic for WAF

END-TO-END SSL:
┌─────────┐      HTTPS       ┌─────────────┐     HTTPS     ┌─────────┐
│ Client  │─────────────────►│ App Gateway │──────────────►│ Backend │
└─────────┘                  └─────────────┘               └─────────┘

USE FOR: Compliance requirements
```

### Web Application Firewall (WAF)
```
WAF PROTECTION:
┌─────────────────────────────────────────────────────────────────┐
│ OWASP TOP 10 PROTECTION:                                       │
│ - SQL Injection                                                │
│ - Cross-Site Scripting (XSS)                                   │
│ - Command Injection                                            │
│ - Local File Inclusion                                         │
│ - Protocol Violations                                          │
│                                                                 │
│ MODES:                                                          │
│ - Detection: Log but don't block                               │
│ - Prevention: Block malicious requests                         │
│                                                                 │
│ CUSTOM RULES:                                                   │
│ - Geo-filtering (block countries)                              │
│ - IP allowlist/blocklist                                       │
│ - Rate limiting                                                │
│ - Header inspection                                            │
└─────────────────────────────────────────────────────────────────┘
```

### Session Affinity
```
COOKIE-BASED AFFINITY:
┌─────────┐                    ┌─────────────┐
│ Client  │───Request 1───────►│ App Gateway │──►  Server A
└─────────┘                    └─────────────┘     (sets cookie)
     │
     │  Cookie: ARRAffinity=ServerA
     │
┌─────────┐                    ┌─────────────┐
│ Client  │───Request 2───────►│ App Gateway │──►  Server A
└─────────┘  (sends cookie)    └─────────────┘     (same server)

USE FOR: Stateful applications that store session locally
```

### Connection Draining
```
SCENARIO: Graceful removal of backend server

WITHOUT DRAINING:
Server removed → Active connections DROPPED → Errors!

WITH DRAINING:
Server marked unhealthy → New connections go elsewhere
                        → Existing connections complete (up to timeout)
                        → Then server removed

SETTING: Connection draining timeout (1-3600 seconds)
```

## CLI Commands

```bash
# Create Application Gateway
az network application-gateway create \
  --resource-group myRG \
  --name myAppGateway \
  --location eastus \
  --sku WAF_v2 \
  --capacity 2 \
  --vnet-name myVNet \
  --subnet AppGatewaySubnet \
  --public-ip-address myAppGwPublicIP \
  --frontend-port 443 \
  --http-settings-port 80 \
  --http-settings-protocol Http \
  --servers 10.0.1.4 10.0.1.5

# Add SSL certificate
az network application-gateway ssl-cert create \
  --resource-group myRG \
  --gateway-name myAppGateway \
  --name myCert \
  --cert-file /path/to/cert.pfx \
  --cert-password "password"

# Create URL path map
az network application-gateway url-path-map create \
  --resource-group myRG \
  --gateway-name myAppGateway \
  --name myPathMap \
  --paths /api/* \
  --address-pool apiBackendPool \
  --http-settings apiHttpSettings \
  --default-address-pool defaultBackendPool \
  --default-http-settings defaultHttpSettings

# Enable WAF
az network application-gateway waf-config set \
  --resource-group myRG \
  --gateway-name myAppGateway \
  --enabled true \
  --firewall-mode Prevention \
  --rule-set-type OWASP \
  --rule-set-version 3.2
```

---

# 3. AZURE FRONT DOOR (Global Layer 7)

## What It Is
Global load balancer and CDN with WAF. Routes traffic to the closest/healthiest backend across regions. Microsoft's global edge network.

## Key Differentiator
```
APPLICATION GATEWAY:
- Regional (single Azure region)
- For backends in same region

FRONT DOOR:
- Global (routes across regions)
- Uses Microsoft's edge network (160+ PoPs)
- Built-in CDN capabilities
- For multi-region deployments
```

## Architecture
```
GLOBAL ROUTING WITH FRONT DOOR:
                         ┌─────────────────────┐
   User in Europe ──────►│  Edge PoP (London)  │
                         └──────────┬──────────┘
                                    │
                         ┌──────────▼──────────┐
                         │    FRONT DOOR       │
                         │  (Global Routing)   │
                         └──────────┬──────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              ▼                     ▼                     ▼
    ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐
    │   West Europe   │   │   East US       │   │   Southeast Asia│
    │   (Closest)     │   │                 │   │                 │
    └─────────────────┘   └─────────────────┘   └─────────────────┘
```

## Tiers

```
FRONT DOOR TIERS:
┌─────────────────────────────────────────────────────────────────┐
│ Standard:                                                       │
│ - Static/dynamic content delivery                              │
│ - Global load balancing                                        │
│ - SSL offload                                                  │
│ - Custom domains                                               │
│                                                                 │
│ Premium:                                                        │
│ - All Standard features                                        │
│ - WAF with managed rules                                       │
│ - Bot protection                                               │
│ - Private Link origin                                          │
│ - Advanced analytics                                           │
└─────────────────────────────────────────────────────────────────┘
```

## Key Features

### Anycast Routing
```
HOW IT WORKS:
- Same IP address announced from all edge locations
- Users automatically route to nearest edge
- No DNS-based latency

User request → Nearest PoP → Optimal backend
```

### Split TCP
```
OPTIMIZATION:
┌─────────┐                  ┌─────────┐                  ┌─────────┐
│ Client  │───Short path────►│  Edge   │───Optimized────►│ Backend │
└─────────┘  (to edge PoP)   └─────────┘  (MS backbone)   └─────────┘

BENEFIT: 
- TCP handshake at edge (low latency)
- Persistent connections to backend
- Faster first byte
```

### Caching
```
CACHE AT EDGE:
┌─────────┐     Cache HIT     ┌─────────┐
│ Client  │◄─────────────────│  Edge   │     (No backend call)
└─────────┘                   └─────────┘

┌─────────┐     Cache MISS    ┌─────────┐                ┌─────────┐
│ Client  │◄─────────────────│  Edge   │◄──────────────│ Backend │
└─────────┘                   └─────────┘  (fetch+cache) └─────────┘

CACHE SETTINGS:
- Query string behavior
- Compression
- Cache duration
```

## CLI Commands

```bash
# Create Front Door profile
az afd profile create \
  --resource-group myRG \
  --profile-name myFrontDoor \
  --sku Premium_AzureFrontDoor

# Create endpoint
az afd endpoint create \
  --resource-group myRG \
  --profile-name myFrontDoor \
  --endpoint-name myEndpoint \
  --enabled-state Enabled

# Create origin group
az afd origin-group create \
  --resource-group myRG \
  --profile-name myFrontDoor \
  --origin-group-name myOriginGroup \
  --probe-request-type GET \
  --probe-protocol Http \
  --probe-path /health

# Add origin
az afd origin create \
  --resource-group myRG \
  --profile-name myFrontDoor \
  --origin-group-name myOriginGroup \
  --origin-name myOrigin \
  --host-name mybackend.azurewebsites.net \
  --http-port 80 \
  --https-port 443 \
  --priority 1 \
  --weight 1000

# Create route
az afd route create \
  --resource-group myRG \
  --profile-name myFrontDoor \
  --endpoint-name myEndpoint \
  --route-name myRoute \
  --origin-group myOriginGroup \
  --supported-protocols Https \
  --patterns-to-match "/*"
```

---

# 4. TRAFFIC MANAGER (DNS-Based)

## What It Is
DNS-based global load balancer. Routes at DNS level - returns IP of best endpoint. No inline data path.

## How It Works
```
DNS-BASED ROUTING:
┌─────────┐                   ┌─────────────────┐
│ Client  │───DNS Query──────►│ Traffic Manager │
│         │   myapp.trafficmanager.net          │
└─────────┘                   └────────┬────────┘
     │                                 │
     │    DNS Response: 20.30.40.50   │
     ◄─────────────────────────────────┘
     │
     │    Direct connection to endpoint
     ▼
┌─────────────────┐
│ Endpoint        │
│ 20.30.40.50     │
└─────────────────┘

KEY POINT: Traffic Manager only handles DNS
           Actual traffic goes directly to endpoint
```

## Routing Methods

### Priority
```
USE CASE: Active-passive failover

Priority 1: East US (Primary)     ──► Used when healthy
Priority 2: West US (Secondary)   ──► Used if primary fails
Priority 3: EU (Tertiary)         ──► Used if both fail
```

### Weighted
```
USE CASE: Load distribution, canary deployments

Endpoint A: Weight 80  ──► Gets 80% of traffic
Endpoint B: Weight 20  ──► Gets 20% of traffic

CANARY EXAMPLE:
Production: Weight 95
New Version: Weight 5  ──► Test with 5% traffic
```

### Performance
```
USE CASE: Lowest latency routing

Traffic Manager measures latency from DNS resolvers to each endpoint
Routes user to endpoint with lowest latency

User in Europe  ──► Europe endpoint (lowest latency)
User in Asia    ──► Asia endpoint (lowest latency)
```

### Geographic
```
USE CASE: Compliance, data residency

Europe users     ──► Europe endpoint (GDPR)
US users         ──► US endpoint
Australia users  ──► Australia endpoint (data sovereignty)
```

### Multivalue
```
USE CASE: Client-side load balancing

Returns multiple healthy endpoint IPs
Client chooses which to connect to

DNS Response: [20.30.40.50, 20.30.40.51, 20.30.40.52]
```

### Subnet
```
USE CASE: Route based on client subnet

10.0.0.0/8    ──► Internal endpoint
0.0.0.0/0     ──► External endpoint
```

## CLI Commands

```bash
# Create Traffic Manager profile
az network traffic-manager profile create \
  --resource-group myRG \
  --name myTrafficManager \
  --routing-method Performance \
  --unique-dns-name myapp \
  --ttl 30 \
  --protocol HTTPS \
  --port 443 \
  --path /health

# Add Azure endpoint
az network traffic-manager endpoint create \
  --resource-group myRG \
  --profile-name myTrafficManager \
  --name eastus-endpoint \
  --type azureEndpoints \
  --target-resource-id /subscriptions/.../publicIPAddresses/eastus-ip \
  --endpoint-status Enabled \
  --priority 1

# Add external endpoint
az network traffic-manager endpoint create \
  --resource-group myRG \
  --profile-name myTrafficManager \
  --name external-endpoint \
  --type externalEndpoints \
  --target external.contoso.com \
  --endpoint-status Enabled \
  --priority 2
```

---

# COMPARISON TABLE

```
┌──────────────────┬─────────────────┬──────────────────┬─────────────────┬─────────────────┐
│ Feature          │ Load Balancer   │ App Gateway      │ Front Door      │ Traffic Manager │
├──────────────────┼─────────────────┼──────────────────┼─────────────────┼─────────────────┤
│ Layer            │ 4 (TCP/UDP)     │ 7 (HTTP/HTTPS)   │ 7 (HTTP/HTTPS)  │ DNS             │
│ Scope            │ Regional        │ Regional         │ Global          │ Global          │
│ Protocol         │ Any             │ HTTP/HTTPS       │ HTTP/HTTPS      │ Any             │
│ SSL Termination  │ ❌              │ ✅               │ ✅              │ ❌              │
│ WAF              │ ❌              │ ✅               │ ✅              │ ❌              │
│ URL Routing      │ ❌              │ ✅               │ ✅              │ ❌              │
│ Caching          │ ❌              │ ❌               │ ✅              │ ❌              │
│ Session Affinity │ Source IP       │ Cookie           │ Cookie          │ ❌              │
│ Inline Traffic   │ ✅              │ ✅               │ ✅              │ ❌ (DNS only)   │
│ Health Probes    │ TCP/HTTP        │ HTTP/HTTPS       │ HTTP/HTTPS      │ HTTP/HTTPS/TCP  │
└──────────────────┴─────────────────┴──────────────────┴─────────────────┴─────────────────┘
```

---

# WHEN TO USE WHAT

```
DECISION TREE:

Need to load balance non-HTTP traffic (TCP/UDP)?
├── YES → Azure Load Balancer
└── NO (HTTP/HTTPS) →
    │
    ├── Need global distribution across regions?
    │   ├── YES → 
    │   │   ├── Need WAF + Caching? → Front Door
    │   │   └── DNS-based only OK? → Traffic Manager
    │   │
    │   └── NO (single region) → Application Gateway
    │
    └── Need WAF in single region? → Application Gateway with WAF

COMMON PATTERNS:

1. Simple web app (single region):
   Internet → Application Gateway → VMs/VMSS

2. Multi-tier app:
   Internet → Public LB → Web tier → Internal LB → App tier

3. Global app (multi-region):
   Internet → Front Door → Regional App Gateways → Backends

4. Active-passive DR:
   Internet → Traffic Manager → Primary Region
                             → Secondary Region (failover)

5. Microservices on AKS:
   Internet → Application Gateway Ingress Controller → AKS pods
```

---

# BEST PRACTICES

```
LOAD BALANCER:
✓ Use Standard SKU for production
✓ Enable zone redundancy
✓ Monitor SNAT port usage
✓ Use NAT Gateway for large outbound workloads
✓ Configure appropriate idle timeout

APPLICATION GATEWAY:
✓ Size subnet appropriately (/24 minimum for v2)
✓ Enable autoscaling
✓ Use WAF in Prevention mode after tuning
✓ Configure connection draining
✓ Use managed certificates when possible

FRONT DOOR:
✓ Use Premium for WAF capabilities
✓ Configure caching rules appropriately
✓ Set up custom domains with proper SSL
✓ Use Private Link for secure origin connections
✓ Monitor cache hit ratio

TRAFFIC MANAGER:
✓ Set appropriate TTL (lower = faster failover, more DNS queries)
✓ Configure health probes correctly
✓ Use nested profiles for complex routing
✓ Consider using with Front Door for comprehensive solution
```
