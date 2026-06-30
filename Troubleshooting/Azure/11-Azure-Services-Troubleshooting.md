# Azure Services Troubleshooting - Extended Guide
## Functions, Container Apps, ACR, Service Bus, API Management

---

# SECTION 1: AZURE FUNCTIONS

---

## 1.1 FUNCTION WON'T TRIGGER

### LEVEL 1 - DIRECT: Trigger Configuration Wrong
```
Error: Function not triggering on HTTP request
```

**Cause is VISIBLE**: Incorrect trigger binding.

**Solution**:
```csharp
// Check function.json or attributes
[FunctionName("MyFunction")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", "post")] HttpRequest req)
{
    // Your code
}
```

---

### LEVEL 1 - DIRECT: Timer Trigger Not Firing
```
Function scheduled for every 5 minutes not running
```

**Check CRON expression**:
```json
{
  "schedule": "0 */5 * * * *"
}
```

**Common mistakes**:
- Azure uses 6-field CRON (includes seconds)
- Missing leading 0 for seconds

---

### LEVEL 2 - INTERMEDIATE: Event Hub/Service Bus Trigger Stuck

**Scenario**: Messages in queue but function not processing.

**Investigation**:
```bash
# Check function host status
az functionapp show -g myRG -n myfunc --query state

# Check if function is disabled
az functionapp function show -g myRG -n myfunc --function-name MyFunction

# Check host.json for batch settings
# Scale might be limited
```

**Common Causes**:
- Function disabled
- Connection string wrong
- Checkpoint blob storage issues (Event Hub)
- Lease expired

---

### LEVEL 3 - COMPLEX: Cold Start Taking Too Long

**Scenario**: First request takes 10+ seconds.

**Hidden Causes**:
- Consumption plan cold start
- Large dependencies
- Slow initialization code

**Solution**:
```bash
# Use Premium plan for pre-warmed instances
az functionapp plan create \
  --name myPremiumPlan \
  --resource-group myRG \
  --sku EP1 \
  --is-linux

# Or keep warm with periodic pings
# Timer trigger every 5 minutes to self-ping
```

---

## 1.2 FUNCTION ERRORS

### LEVEL 1 - DIRECT: Out of Memory
```
Error: System.OutOfMemoryException
```

**Cause**: Function exceeds memory limit.

**Solution**:
```bash
# Consumption: 1.5 GB max
# Premium EP1: 3.5 GB
# Premium EP2: 7 GB
# Premium EP3: 14 GB

# Upgrade to Premium
az functionapp plan update --name myPlan --sku EP2
```

---

### LEVEL 2 - INTERMEDIATE: Timeout Errors
```
Error: Function execution timeout
```

**Limits**:
```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Consumption Plan:  5 min (default), 10 min (max)               â”‚
â”‚ Premium Plan:      30 min (default), unlimited configurable    â”‚
â”‚ Dedicated Plan:    30 min (default), unlimited configurable    â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

**Solution**:
```json
// host.json
{
  "functionTimeout": "00:10:00"
}
```

---

# SECTION 2: AZURE CONTAINER APPS

---

## 2.1 CONTAINER WON'T START

### LEVEL 1 - DIRECT: Image Pull Failed
```
Error: Failed to pull image
```

**Investigation**:
```bash
# Check container app status
az containerapp show -g myRG -n myapp --query "properties.provisioningState"

# Check logs
az containerapp logs show -g myRG -n myapp --type system
```

**Solution**:
```bash
# For ACR, ensure managed identity has AcrPull role
az containerapp registry set \
  -g myRG -n myapp \
  --server myacr.azurecr.io \
  --identity system
```

---

### LEVEL 2 - INTERMEDIATE: Container Keeps Restarting

**Investigation**:
```bash
# Check container logs
az containerapp logs show -g myRG -n myapp --type console

# Check replica status
az containerapp replica list -g myRG -n myapp
```

**Common Causes**:
- Application crash on startup
- Health probe failing
- Missing environment variables

---

### LEVEL 3 - COMPLEX: Scaling Not Working

**Scenario**: App not scaling despite load.

**Investigation**:
```bash
# Check scaling rules
az containerapp show -g myRG -n myapp --query "properties.template.scale"

# Check current replicas
az containerapp replica list -g myRG -n myapp

# Check metrics
# Portal â†’ Container App â†’ Metrics â†’ Requests/Replica Count
```

**Solution**:
```bash
# Configure HTTP scaling
az containerapp update -g myRG -n myapp \
  --min-replicas 1 \
  --max-replicas 10 \
  --scale-rule-name http-scaling \
  --scale-rule-type http \
  --scale-rule-http-concurrency 100
```

---

# SECTION 3: AZURE CONTAINER REGISTRY (ACR)

---

## 3.1 PUSH/PULL FAILURES

### LEVEL 1 - DIRECT: Authentication Failed
```
Error: unauthorized: authentication required
```

**Solution**:
```bash
# Login to ACR
az acr login --name myacr

# Or use service principal
docker login myacr.azurecr.io -u <sp-id> -p <sp-secret>

# For AKS, attach ACR
az aks update -g myRG -n myAKS --attach-acr myacr
```

---

### LEVEL 1 - DIRECT: Image Not Found
```
Error: manifest not found
```

**Investigation**:
```bash
# List repositories
az acr repository list --name myacr

# List tags
az acr repository show-tags --name myacr --repository myapp
```

---

### LEVEL 2 - INTERMEDIATE: Push Denied - Quota Exceeded
```
Error: denied: requested access to the resource is denied
```

**Causes**:
- Storage quota exceeded
- Repository limit reached (Basic: 2 repos)

**Solution**:
```bash
# Check usage
az acr show-usage --name myacr

# Upgrade SKU
az acr update --name myacr --sku Standard

# Or purge old images
az acr purge --filter "myapp:.*" --ago 30d --keep 5 --dry-run
az acr purge --filter "myapp:.*" --ago 30d --keep 5
```

---

### LEVEL 3 - COMPLEX: Geo-Replication Sync Issues

**Scenario**: Images available in one region but not another.

**Investigation**:
```bash
# Check replication status
az acr replication list --registry myacr

# Check specific replication
az acr replication show --registry myacr --name westus
```

**Solution**:
```bash
# Wait for sync (usually < 1 minute)
# Or re-push image

# Check webhook for replication events
az acr webhook list --registry myacr
```

---

# SECTION 4: AZURE SERVICE BUS

---

## 4.1 MESSAGE DELIVERY ISSUES

### LEVEL 1 - DIRECT: Queue/Topic Not Found
```
Error: The messaging entity could not be found
```

**Solution**:
```bash
# List queues
az servicebus queue list --namespace-name mybus -g myRG

# Create queue
az servicebus queue create --namespace-name mybus -g myRG --name myqueue
```

---

### LEVEL 1 - DIRECT: Unauthorized
```
Error: Unauthorized access
```

**Investigation**:
```bash
# Check SAS policy
az servicebus namespace authorization-rule list \
  --namespace-name mybus -g myRG

# Get connection string
az servicebus namespace authorization-rule keys list \
  --namespace-name mybus -g myRG \
  --name RootManageSharedAccessKey
```

---

### LEVEL 2 - INTERMEDIATE: Messages Going to Dead Letter

**Investigation**:
```bash
# Check dead letter queue
az servicebus queue show --namespace-name mybus -g myRG --name myqueue \
  --query "countDetails.deadLetterMessageCount"

# Peek dead letter messages (use Service Bus Explorer or SDK)
```

**Common Causes**:
- MaxDeliveryCount exceeded (default: 10)
- Message TTL expired
- Message too large
- Processing exception

**Solution**:
```bash
# Increase max delivery count
az servicebus queue update \
  --namespace-name mybus -g myRG --name myqueue \
  --max-delivery-count 20

# Process dead letter queue separately
```

---

### LEVEL 3 - COMPLEX: Messages Stuck - Not Being Delivered

**Scenario**: Messages in queue but not being received.

**Hidden Causes**:
- Receiver not connected
- Session lock (for session-enabled queues)
- Prefetch buffer full
- Competing consumers with long lock

**Investigation**:
```bash
# Check active message count
az servicebus queue show --namespace-name mybus -g myRG --name myqueue \
  --query "countDetails.activeMessageCount"

# Check if queue requires sessions
az servicebus queue show --namespace-name mybus -g myRG --name myqueue \
  --query "requiresSession"
```

---

## 4.2 PERFORMANCE ISSUES

### LEVEL 2 - INTERMEDIATE: Throttling
```
Error: The request was throttled
```

**Cause**: Exceeded messaging units.

**Solution**:
```bash
# Premium: Increase messaging units
az servicebus namespace update \
  --name mybus -g myRG \
  --capacity 2  # 2 messaging units
```

---

# SECTION 5: API MANAGEMENT

---

## 5.1 API NOT RESPONDING

### LEVEL 1 - DIRECT: 404 Not Found
```
Error: Resource not found
```

**Investigation**:
```bash
# List APIs
az apim api list -g myRG --service-name myapim

# Check API path
az apim api show -g myRG --service-name myapim --api-id myapi \
  --query "path"
```

---

### LEVEL 1 - DIRECT: 401 Unauthorized
```
Error: Access denied due to missing subscription key
```

**Solution**:
```bash
# Get subscription key
az apim subscription list -g myRG --service-name myapim

# Call with key
curl -H "Ocp-Apim-Subscription-Key: <key>" https://myapim.azure-api.net/myapi/resource
```

---

### LEVEL 2 - INTERMEDIATE: 500 Backend Error
```
Error: Backend service returned error
```

**Investigation**:
```bash
# Check backend URL
az apim backend list -g myRG --service-name myapim

# Test backend directly
curl https://backend.example.com/api/resource

# Check APIM trace
# Enable in Portal: APIs â†’ Test â†’ Trace
```

**Common Causes**:
- Backend URL wrong
- Backend not reachable from APIM VNet
- Backend SSL certificate issue
- Backend timeout

---

### LEVEL 3 - COMPLEX: Intermittent Timeouts

**Scenario**: Random timeouts despite backend being healthy.

**Hidden Causes**:
- APIM timeout too short (default: 30s)
- Backend slow under load
- Connection pool exhaustion
- APIM gateway capacity

**Investigation**:
```bash
# Check APIM metrics
# Portal â†’ APIM â†’ Metrics â†’ Backend Duration, Failed Requests

# Check policy timeout
# <forward-request timeout="60" />
```

**Solution**:
```xml
<!-- Increase timeout in policy -->
<backend>
    <forward-request timeout="60" />
</backend>
```

---

## 5.2 POLICY ISSUES

### LEVEL 2 - INTERMEDIATE: Policy Not Applied
```
Expected header transformation not happening
```

**Investigation**:
- Check policy scope (All APIs vs specific API vs operation)
- Check policy order (inbound â†’ backend â†’ outbound)
- Use trace to see policy execution

**Example Policy Debug**:
```xml
<policies>
    <inbound>
        <trace source="policy">
            <message>Inbound processing</message>
        </trace>
        <set-header name="X-Custom" exists-action="override">
            <value>my-value</value>
        </set-header>
    </inbound>
</policies>
```

---

# QUICK REFERENCE

```
COMMON ERRORS:

Azure Functions:
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Cold start slow        â†’ Use Premium plan                     â”‚
â”‚ Timeout error          â†’ Increase functionTimeout             â”‚
â”‚ Out of memory          â†’ Upgrade plan, optimize code          â”‚
â”‚ Trigger not firing     â†’ Check bindings, connection strings   â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜

Container Apps:
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Image pull failed      â†’ Check registry auth, image exists    â”‚
â”‚ Container restarting   â†’ Check logs, health probes            â”‚
â”‚ Not scaling            â†’ Check scale rules, metrics           â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜

ACR:
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Auth failed            â†’ az acr login, check SP credentials   â”‚
â”‚ Image not found        â†’ Check repo/tag exists                â”‚
â”‚ Quota exceeded         â†’ Upgrade SKU or purge old images      â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜

Service Bus:
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Entity not found       â†’ Check queue/topic exists             â”‚
â”‚ Messages dead-lettered â†’ Check MaxDeliveryCount, exceptions   â”‚
â”‚ Throttling             â†’ Upgrade to Premium, add MUs          â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜

API Management:
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ 404 Not Found          â†’ Check API path configuration         â”‚
â”‚ 401 Unauthorized       â†’ Include subscription key             â”‚
â”‚ 500 Backend Error      â†’ Check backend URL, connectivity      â”‚
â”‚ Timeout                â†’ Increase forward-request timeout     â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```
