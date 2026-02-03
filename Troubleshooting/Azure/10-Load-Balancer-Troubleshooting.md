# Azure Load Balancer Troubleshooting - Complete Guide
## Load Balancer, Application Gateway, Front Door, Traffic Manager

---

# SECTION 1: AZURE LOAD BALANCER

---

## 1.1 BACKEND POOL UNHEALTHY

### LEVEL 1 - DIRECT: Health Probe Port Closed

**Scenario**: All backends show as unhealthy in LB.

```
Portal shows: Backend pool health = 0%
```

**Cause is VISIBLE**: The probe port is not open on backend VMs.

**Investigation**:
```bash
# Check probe configuration
az network lb probe show -g myRG --lb-name myLB -n myProbe

# Check if port is listening on VM
# SSH to VM and run:
ss -tlnp | grep 80

# Check NSG allows probe
az network nsg rule list -g myRG --nsg-name myNSG -o table
```

**Solution**:
```bash
# Ensure application is listening on probe port
# Ensure NSG allows Azure Load Balancer tag
az network nsg rule create \
  -g myRG --nsg-name myNSG \
  -n AllowAzureLB \
  --priority 100 \
  --source-address-prefixes AzureLoadBalancer \
  --destination-port-ranges 80 \
  --access Allow
```

---

### LEVEL 1 - DIRECT: Health Probe Path Returns Non-200

**Scenario**: HTTP probe configured but backends unhealthy.

```
Probe path: /health
Response: 404 Not Found
```

**Cause is VISIBLE**: Probe path doesn't exist or returns error.

**Solution**:
```bash
# Test from VM itself
curl -v http://localhost/health

# Fix application to return 200 on probe path
# Or change probe path to valid endpoint
az network lb probe update \
  -g myRG --lb-name myLB -n myProbe \
  --path /
```

---

### LEVEL 2 - INTERMEDIATE: Intermittent Health Failures

**Scenario**: Backends flip between healthy/unhealthy.

**Investigation**:
```bash
# Check probe settings
az network lb probe show -g myRG --lb-name myLB -n myProbe

# On VM, monitor probe responses
sudo tcpdump -i any port 80 -nn

# Check application logs during failures
journalctl -u myapp --since "10 minutes ago"
```

**Common Causes**:
- Application too slow to respond within probe timeout
- Resource pressure on VM (CPU/memory)
- Application restarting

**Solution**:
```bash
# Increase probe interval and threshold
az network lb probe update \
  -g myRG --lb-name myLB -n myProbe \
  --interval 15 \
  --threshold 3
```

---

### LEVEL 3 - COMPLEX: Healthy Backends But No Traffic

**Scenario**: Health probes pass but no application traffic reaches backends.

**Hidden Causes**:
- No load balancing rule configured
- Rule frontend port different from what clients use
- DSR (Direct Server Return) / Floating IP misconfiguration
- Backend application listening on wrong IP

**Deep Investigation**:
```bash
# Check LB rules exist
az network lb rule list -g myRG --lb-name myLB -o table

# Verify rule maps frontend to backend correctly
az network lb rule show -g myRG --lb-name myLB -n myRule

# Check frontend IP
az network lb frontend-ip list -g myRG --lb-name myLB

# On VM, check what IP app is listening on
ss -tlnp

# If floating IP enabled, app must listen on frontend IP too
# Check if loopback adapter configured for floating IP
ip addr show lo
```

**Solution for Floating IP**:
```bash
# On Windows backend, add loopback adapter with frontend IP
# On Linux:
sudo ip addr add 20.30.40.50/32 dev lo
```

---

## 1.2 SNAT EXHAUSTION

### LEVEL 2 - INTERMEDIATE: Outbound Connections Failing

**Scenario**: Applications intermittently fail to make outbound connections.

```
Error: Connection timed out
Error: Cannot assign requested address
```

**Investigation**:
```bash
# Check SNAT port usage in Azure Monitor
# Metrics → Load Balancer → SNAT Connection Count

# On VM, check connection count
ss -s
netstat -an | wc -l

# Check allocated ports per instance
az network lb show -g myRG -n myLB --query "outboundRules"
```

**Understanding SNAT**:
```
SNAT PORT ALLOCATION:
┌──────────────────────────────────────────────────────────────┐
│ Each Public IP = 64,000 ports                               │
│ Ports divided among backend pool instances                  │
│                                                              │
│ Example: 1 IP, 10 VMs = 6,400 ports each                    │
│ If VM makes 7,000 concurrent connections → EXHAUSTED!       │
└──────────────────────────────────────────────────────────────┘
```

**Solution**:
```bash
# Option 1: Add more frontend IPs
az network public-ip create -g myRG -n myPIP2 --sku Standard
az network lb frontend-ip create \
  -g myRG --lb-name myLB \
  -n frontend2 \
  --public-ip-address myPIP2

# Option 2: Configure explicit outbound rule with more ports
az network lb outbound-rule create \
  -g myRG --lb-name myLB \
  -n myOutboundRule \
  --frontend-ip-configs frontend1 frontend2 \
  --address-pool myBackendPool \
  --allocated-outbound-ports 10000

# Option 3: Use NAT Gateway (recommended for high outbound)
az network nat gateway create \
  -g myRG -n myNATGateway \
  --public-ip-addresses myNATIP
```

---

### LEVEL 3 - COMPLEX: SNAT with Internal LB Only

**Scenario**: VMs behind Internal LB cannot reach internet.

**Hidden Cause**: Internal LB has no SNAT capability.

**Investigation**:
```bash
# Check LB type
az network lb show -g myRG -n myLB --query "frontendIpConfigurations[0].publicIpAddress"
# If null → Internal LB

# VMs have no public IP and no outbound path
az vm show -g myRG -n myVM --query "networkProfile.networkInterfaces[0]"
```

**Solution**:
```bash
# Option 1: Add NAT Gateway to subnet
az network nat gateway create -g myRG -n myNATGW --public-ip-addresses natPIP
az network vnet subnet update \
  -g myRG --vnet-name myVNet -n mySubnet \
  --nat-gateway myNATGW

# Option 2: Add public IP to VMs (not recommended)

# Option 3: Use Azure Firewall for outbound
```

---

## 1.3 TRAFFIC DISTRIBUTION ISSUES

### LEVEL 2 - INTERMEDIATE: Uneven Load Distribution

**Scenario**: One backend gets more traffic than others.

**Investigation**:
```bash
# Check distribution mode
az network lb rule show -g myRG --lb-name myLB -n myRule \
  --query "loadDistribution"

# Check if backends have different weights (VMSS)
# Standard LB doesn't support weights, all equal

# Monitor per-instance metrics
# Azure Monitor → Metrics → Split by Backend IP
```

**Common Causes**:
- Source IP affinity enabled (same clients → same server)
- Long-lived connections (WebSocket, database)
- Unequal backend capacity

**Solution**:
```bash
# Change to 5-tuple distribution
az network lb rule update \
  -g myRG --lb-name myLB -n myRule \
  --load-distribution Default

# For long-lived connections, consider connection draining
```

---

# SECTION 2: APPLICATION GATEWAY

---

## 2.1 502 BAD GATEWAY

### LEVEL 1 - DIRECT: Backend Not Responding

**Scenario**: Users get 502 errors.

```
HTTP 502 Bad Gateway
```

**Cause is VISIBLE**: Backend server not responding.

**Investigation**:
```bash
# Check backend health
az network application-gateway show-backend-health \
  -g myRG -n myAppGW

# Check if backend is reachable from App GW subnet
# App GW needs network path to backend

# Check backend is listening
curl -v http://backend-ip:port/
```

**Solution**:
```bash
# Ensure NSG allows App GW subnet
# Ensure backend application is running
# Check HTTP settings match backend configuration
az network application-gateway http-settings show \
  -g myRG --gateway-name myAppGW -n myHTTPSettings
```

---

### LEVEL 2 - INTERMEDIATE: 502 After Working

**Scenario**: Was working, suddenly 502 errors.

**Investigation**:
```bash
# Check backend health history
# Azure Monitor → Application Gateway → Backend health

# Check if SSL cert expired (for HTTPS backends)
az network application-gateway ssl-cert list -g myRG --gateway-name myAppGW

# Check if backend was redeployed (new IP)
# App GW may have old IP cached
```

**Common Causes**:
- Backend scaled in (IP changed)
- SSL certificate expired
- Backend timeout
- Network route changed

**Solution**:
```bash
# Update backend pool
az network application-gateway address-pool update \
  -g myRG --gateway-name myAppGW -n myPool \
  --servers 10.0.1.5 10.0.1.6

# Increase backend timeout
az network application-gateway http-settings update \
  -g myRG --gateway-name myAppGW -n myHTTPSettings \
  --timeout 60
```

---

### LEVEL 3 - COMPLEX: Intermittent 502s Under Load

**Scenario**: 502 errors only during traffic spikes.

**Hidden Causes**:
- App GW capacity exhausted
- Backend connection limits
- Backend thread pool exhausted

**Deep Investigation**:
```bash
# Check App GW capacity units
# Azure Monitor → Metrics → Capacity Units

# Check failed requests metric
# Azure Monitor → Metrics → Failed Requests

# Check if autoscaling is working
az network application-gateway show -g myRG -n myAppGW \
  --query "autoscaleConfiguration"

# Backend metrics
# Check CPU, memory, connection count on backend
```

**Solution**:
```bash
# Enable autoscaling
az network application-gateway update \
  -g myRG -n myAppGW \
  --min-capacity 2 \
  --max-capacity 10

# Increase backend pool size
# Add more backend instances
```

---

## 2.2 WAF BLOCKING LEGITIMATE TRAFFIC

### LEVEL 1 - DIRECT: False Positive Blocks

**Scenario**: WAF blocks legitimate requests.

```
HTTP 403 Forbidden
WAF Rule ID: 942100 (SQL Injection)
```

**Cause is VISIBLE**: WAF rule triggered on legitimate content.

**Investigation**:
```bash
# Check WAF logs
# Log Analytics query:
AzureDiagnostics
| where ResourceType == "APPLICATIONGATEWAYS"
| where action_s == "Blocked"
| project TimeGenerated, requestUri_s, ruleId_s, Message
```

**Solution**:
```bash
# Option 1: Create exclusion for specific rule
az network application-gateway waf-config set \
  -g myRG --gateway-name myAppGW \
  --enabled true \
  --firewall-mode Prevention \
  --rule-set-type OWASP \
  --rule-set-version 3.2 \
  --exclusion RequestBodyCheck Equals "commentField"

# Option 2: Disable specific rule
az network application-gateway waf-config set \
  -g myRG --gateway-name myAppGW \
  --enabled true \
  --disabled-rules 942100
```

---

### LEVEL 2 - INTERMEDIATE: WAF Blocks File Uploads

**Scenario**: Large file uploads blocked.

**Investigation**:
```bash
# Check request body size limits
# Default max request body: 128 KB for WAF

# Check WAF logs for size violations
```

**Solution**:
```bash
# Increase request body limit
az network application-gateway waf-policy update \
  -g myRG -n myWAFPolicy \
  --request-body-check true \
  --max-request-body-size-in-kb 1024 \
  --file-upload-limit-in-mb 100
```

---

## 2.3 SSL/TLS ISSUES

### LEVEL 1 - DIRECT: SSL Handshake Failure

**Scenario**: HTTPS connections fail.

```
Error: SSL handshake failed
```

**Investigation**:
```bash
# Check certificate
az network application-gateway ssl-cert list -g myRG --gateway-name myAppGW

# Check certificate validity
openssl s_client -connect myappgw.eastus.cloudapp.azure.com:443

# Check if cert is for correct domain
openssl x509 -in cert.pem -text | grep "Subject:"
```

**Solution**:
```bash
# Update certificate
az network application-gateway ssl-cert update \
  -g myRG --gateway-name myAppGW -n myCert \
  --cert-file new-cert.pfx \
  --cert-password "password"
```

---

### LEVEL 2 - INTERMEDIATE: Certificate Chain Incomplete

**Scenario**: Some clients fail, others work.

**Hidden Cause**: Intermediate certificate missing.

**Investigation**:
```bash
# Test certificate chain
openssl s_client -connect myapp.com:443 -showcerts

# Check for chain errors
# "unable to get local issuer certificate" = missing intermediate
```

**Solution**:
```bash
# Create PFX with full chain
# Include: server cert + intermediate cert + root cert
openssl pkcs12 -export \
  -out fullchain.pfx \
  -inkey server.key \
  -in server.crt \
  -certfile intermediate.crt
```

---

# SECTION 3: FRONT DOOR

---

## 3.1 ORIGIN NOT REACHABLE

### LEVEL 1 - DIRECT: Origin Health Probe Failing

**Scenario**: Front Door can't reach origin.

**Investigation**:
```bash
# Check origin health
az afd origin show \
  -g myRG --profile-name myFD \
  --origin-group-name myGroup \
  --origin-name myOrigin

# Ensure origin is accessible from internet
# Front Door connects from Microsoft network
curl -H "Host: myapp.azurefd.net" https://origin.example.com
```

**Solution**:
```bash
# Check origin firewall allows Front Door IPs
# Use AzureFrontDoor.Backend service tag

# Or use Private Link origin (Premium)
az afd origin update \
  -g myRG --profile-name myFD \
  --origin-group-name myGroup \
  --origin-name myOrigin \
  --enable-private-link true
```

---

### LEVEL 2 - INTERMEDIATE: Caching Issues

**Scenario**: Old content served despite origin update.

**Investigation**:
```bash
# Check cache behavior
az afd route show -g myRG --profile-name myFD \
  --endpoint-name myEndpoint --route-name myRoute \
  --query "cacheConfiguration"

# Check cache hit ratio in metrics
```

**Solution**:
```bash
# Purge cache
az afd endpoint purge \
  -g myRG --profile-name myFD \
  --endpoint-name myEndpoint \
  --content-paths "/*"

# Disable caching for dynamic content
az afd route update \
  -g myRG --profile-name myFD \
  --endpoint-name myEndpoint \
  --route-name myRoute \
  --enable-caching false
```

---

## 3.2 CUSTOM DOMAIN ISSUES

### LEVEL 1 - DIRECT: Custom Domain Not Working

**Scenario**: Custom domain returns error.

**Investigation**:
```bash
# Check domain validation status
az afd custom-domain show \
  -g myRG --profile-name myFD \
  --custom-domain-name myDomain

# Check DNS CNAME
nslookup myapp.contoso.com
# Should point to: myendpoint.azurefd.net
```

**Solution**:
```bash
# Add CNAME record in DNS:
# myapp.contoso.com → myendpoint.azurefd.net

# Validate domain
az afd custom-domain create \
  -g myRG --profile-name myFD \
  --custom-domain-name myDomain \
  --host-name myapp.contoso.com
```

---

# SECTION 4: TRAFFIC MANAGER

---

## 4.1 ENDPOINT SHOWS DEGRADED

### LEVEL 1 - DIRECT: Health Check Failing

**Scenario**: Endpoint status is Degraded.

**Investigation**:
```bash
# Check endpoint status
az network traffic-manager endpoint show \
  -g myRG --profile-name myTM \
  --type azureEndpoints -n myEndpoint

# Check probe settings
az network traffic-manager profile show -g myRG -n myTM \
  --query "monitorConfig"

# Test endpoint manually
curl -v https://myapp.azurewebsites.net/health
```

**Solution**:
```bash
# Fix application health endpoint
# Update probe path
az network traffic-manager profile update \
  -g myRG -n myTM \
  --path /health \
  --protocol HTTPS \
  --port 443
```

---

### LEVEL 2 - INTERMEDIATE: Slow Failover

**Scenario**: Takes too long to failover when endpoint fails.

**Understanding**:
```
FAILOVER TIME = (Probe Interval × Tolerated Failures) + DNS TTL

Default: (30s × 3) + 60s = 150 seconds minimum!
```

**Solution**:
```bash
# Reduce probe interval
az network traffic-manager profile update \
  -g myRG -n myTM \
  --interval 10 \
  --timeout 5 \
  --tolerated-number-of-failures 2 \
  --ttl 30

# New failover time: (10 × 2) + 30 = 50 seconds
```

---

### LEVEL 3 - COMPLEX: DNS Caching Prevents Failover

**Scenario**: Failover happened but clients still hit old endpoint.

**Hidden Cause**: DNS resolvers caching beyond TTL.

**Investigation**:
```bash
# Check TTL setting
az network traffic-manager profile show -g myRG -n myTM \
  --query "dnsConfig.ttl"

# Check what resolvers are caching
dig myapp.trafficmanager.net

# Some resolvers ignore TTL (especially corporate)
```

**Solution**:
```bash
# Lower TTL (balance with DNS query costs)
az network traffic-manager profile update -g myRG -n myTM --ttl 30

# Client-side: Flush DNS cache
# Windows: ipconfig /flushdns
# Linux: systemd-resolve --flush-caches

# Long-term: Consider Front Door for faster failover
# (anycast-based, no DNS propagation delay)
```

---

# QUICK REFERENCE

```
COMMON ERRORS → LIKELY CAUSE:

Load Balancer:
┌────────────────────────────────────────────────────────────────┐
│ All backends unhealthy     → NSG blocking, probe port closed  │
│ Intermittent health flips  → App slow, resource pressure      │
│ Outbound connection fails  → SNAT exhaustion                  │
│ No traffic distribution    → No LB rule configured            │
└────────────────────────────────────────────────────────────────┘

Application Gateway:
┌────────────────────────────────────────────────────────────────┐
│ 502 Bad Gateway            → Backend unreachable/unhealthy    │
│ 504 Gateway Timeout        → Backend too slow                 │
│ 403 Forbidden              → WAF blocking                     │
│ SSL handshake failed       → Certificate issue                │
└────────────────────────────────────────────────────────────────┘

Front Door:
┌────────────────────────────────────────────────────────────────┐
│ Origin unhealthy           → Origin firewall, wrong path      │
│ Stale content              → Caching, need purge              │
│ Custom domain error        → DNS not configured               │
└────────────────────────────────────────────────────────────────┘

Traffic Manager:
┌────────────────────────────────────────────────────────────────┐
│ Endpoint degraded          → Health probe failing             │
│ Slow failover              → High TTL, probe interval         │
│ Wrong endpoint returned    → DNS caching                      │
└────────────────────────────────────────────────────────────────┘
```

---

# DIAGNOSTIC COMMANDS

```bash
# Load Balancer
az network lb show-backend-health -g myRG -n myLB

# Application Gateway
az network application-gateway show-backend-health -g myRG -n myAppGW

# Front Door
az afd origin show -g myRG --profile-name myFD --origin-group-name myGroup --origin-name myOrigin

# Traffic Manager
az network traffic-manager endpoint show -g myRG --profile-name myTM --type azureEndpoints -n myEndpoint

# Test from different locations
# Use Azure Network Watcher or third-party tools
az network watcher test-connectivity \
  -g myRG --source-resource myVM \
  --dest-address 10.0.1.4 --dest-port 80
```
