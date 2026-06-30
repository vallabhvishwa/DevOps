# Azure Cloud Troubleshooting - Complete Guide
## All Major Azure Services - From Basic to Advanced

---

# SECTION 1: VIRTUAL MACHINES

---

## 1.1 VM WON'T START

### LEVEL 1 - DIRECT: Quota Exceeded
```
Error: OperationNotAllowed - Operation could not be completed as it 
results in exceeding approved Total Regional Cores quota.
```

**Cause is VISIBLE**: Not enough vCPU quota.

**Solution**:
```bash
# Check quota
az vm list-usage --location eastus -o table

# Request increase via portal or support ticket
```

---

### LEVEL 1 - DIRECT: Disk Not Found
```
Error: Disk 'myDisk' not found
```

**Cause is VISIBLE**: Referenced disk doesn't exist.

**Solution**:
```bash
# List disks
az disk list -g myRG -o table

# Attach correct disk
az vm disk attach -g myRG --vm-name myVM --name correctDisk
```

---

### LEVEL 2 - INTERMEDIATE: VM Stuck in Creating

**Scenario**: VM shows "Creating" for long time.

**Investigation**:
```bash
# Check deployment
az deployment group list -g myRG -o table

# Check activity log
az monitor activity-log list -g myRG --offset 1h

# Check for extension issues
az vm extension list -g myRG --vm-name myVM
```

---

### LEVEL 3 - COMPLEX: VM Boots But Can't Connect

**Scenario**: VM is running but SSH/RDP fails.

**Hidden Causes**:
- NSG blocking port 22/3389
- No public IP or wrong IP
- OS firewall
- Network route issue
- Boot failure inside VM

**Deep Investigation**:
```bash
# Check NSG rules
az network nsg rule list -g myRG --nsg-name myNSG -o table

# Check public IP
az vm show -g myRG -n myVM --show-details --query publicIps

# Use serial console (Portal)
# VM > Serial console

# Use boot diagnostics
az vm boot-diagnostics get-boot-log -g myRG -n myVM

# Reset SSH/RDP (last resort)
az vm user reset-ssh -g myRG -n myVM
```

---

# SECTION 2: AKS (KUBERNETES)

---

## 2.1 CLUSTER ISSUES

### LEVEL 1 - DIRECT: Cluster Creation Failed
```
Error: Subnet is not within the VNet address space
```

**Cause is VISIBLE**: Subnet CIDR doesn't fit in VNet.

**Solution**: Fix subnet configuration.

---

### LEVEL 2 - INTERMEDIATE: Nodes Not Ready

**Scenario**: `kubectl get nodes` shows NotReady.

```bash
# Check node status
kubectl describe node <node-name>

# Check kubelet logs (via Azure)
az aks nodepool get-upgrade-profile -g myRG --cluster-name myAKS

# Check for VM issues
az vmss list-instances -g MC_myRG_myAKS_eastus --name aks-nodepool1
```

**Common Causes**:
- Disk pressure
- Memory pressure
- Network unreachable
- kubelet crashed

---

### LEVEL 3 - COMPLEX: Intermittent DNS Failures

**Scenario**: Some pods can resolve DNS, some can't.

**Hidden Causes**:
- CoreDNS pods not on all nodes
- Network policy blocking
- CoreDNS overloaded

**Deep Investigation**:
```bash
# Check CoreDNS pods
kubectl get pods -n kube-system -l k8s-app=kube-dns -o wide

# Check CoreDNS logs
kubectl logs -n kube-system -l k8s-app=kube-dns

# Test from affected pod
kubectl exec <pod> -- nslookup kubernetes.default

# Check HPA on CoreDNS
kubectl get hpa -n kube-system coredns
```

---

## 2.2 APPLICATION ISSUES ON AKS

### LEVEL 1 - DIRECT: ImagePullBackOff
```
Warning: Failed to pull image "myacr.azurecr.io/app:v1"
unauthorized: authentication required
```

**Cause is VISIBLE**: AKS can't authenticate to ACR.

**Solution**:
```bash
# Attach ACR to AKS
az aks update -g myRG -n myAKS --attach-acr myacr

# Or create pull secret
kubectl create secret docker-registry acr-secret \
  --docker-server=myacr.azurecr.io \
  --docker-username=<acr-user> \
  --docker-password=<acr-pass>
```

---

### LEVEL 1 - DIRECT: Pod OOMKilled
```
Last State: Terminated
Reason: OOMKilled
```

**Cause is VISIBLE**: Container exceeded memory limit.

**Solution**:
```yaml
resources:
  limits:
    memory: "1Gi"  # Increase limit
  requests:
    memory: "512Mi"
```

---

# SECTION 3: STORAGE

---

## 3.1 BLOB STORAGE ISSUES

### LEVEL 1 - DIRECT: Access Denied
```
Error: This request is not authorized
```

**Cause is VISIBLE**: Missing RBAC or SAS.

**Solution**:
```bash
# Check permissions
az role assignment list --scope /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Storage/storageAccounts/<account>

# Assign role
az role assignment create --assignee <principal-id> --role "Storage Blob Data Contributor" --scope <storage-account-id>
```

---

### LEVEL 1 - DIRECT: Blob Not Found
```
Error: The specified blob does not exist
```

**Cause is VISIBLE**: Wrong container or blob name.

**Solution**:
```bash
# List blobs
az storage blob list --account-name mystorageaccount --container-name mycontainer -o table
```

---

### LEVEL 2 - INTERMEDIATE: Storage Throttled

**Scenario**: Operations failing intermittently with HTTP 503.

**Investigation**:
```bash
# Check metrics in portal
# Storage account > Metrics > Transactions > SuccessServerOther
# Split by ResponseType to see throttling

# Check account limits
# Premium: Higher IOPS
# Standard: 20,000 requests/second
```

---

## 3.2 AZURE FILES ISSUES

### LEVEL 1 - DIRECT: Mount Failed
```
mount error(13): Permission denied
```

**Cause is VISIBLE**: Wrong credentials.

**Solution**:
```bash
# Get correct credentials
az storage account keys list -g myRG -n mystorageaccount

# Mount with correct key
sudo mount -t cifs //mystorageaccount.file.core.windows.net/myshare /mnt/myshare -o username=mystorageaccount,password=<key>
```

---

# SECTION 4: DATABASES

---

## 4.1 AZURE SQL ISSUES

### LEVEL 1 - DIRECT: Firewall Blocking
```
Error: Cannot open server 'myserver' requested by the login. 
Client with IP address 'x.x.x.x' is not allowed to access the server.
```

**Cause is VISIBLE**: IP not in firewall rules.

**Solution**:
```bash
# Add firewall rule
az sql server firewall-rule create -g myRG -s myserver -n AllowMyIP --start-ip-address x.x.x.x --end-ip-address x.x.x.x
```

---

### LEVEL 1 - DIRECT: Login Failed
```
Error: Login failed for user 'myuser'
```

**Cause is VISIBLE**: Wrong credentials.

**Solution**:
```bash
# Reset password
az sql server update -g myRG -n myserver --admin-password 'NewPassword123!'
```

---

### LEVEL 2 - INTERMEDIATE: Database Slow

**Investigation**:
```sql
-- Check expensive queries
SELECT TOP 10 
    total_worker_time/execution_count AS avg_cpu,
    total_logical_reads/execution_count AS avg_reads,
    SUBSTRING(text, 1, 100) AS query_text
FROM sys.dm_exec_query_stats 
CROSS APPLY sys.dm_exec_sql_text(sql_handle)
ORDER BY avg_cpu DESC;

-- Check blocking
SELECT * FROM sys.dm_exec_requests WHERE blocking_session_id > 0;
```

---

## 4.2 COSMOS DB ISSUES

### LEVEL 1 - DIRECT: Request Rate Too Large
```
Error: Request rate is large. Please retry after sometime.
```

**Cause is VISIBLE**: Exceeding RU/s.

**Solution**:
```bash
# Increase throughput
az cosmosdb sql container throughput update \
  --account-name mycosmosaccount \
  --database-name mydb \
  --name mycontainer \
  --resource-group myRG \
  --throughput 1000
```

---

# SECTION 5: NETWORKING

---

## 5.1 CONNECTIVITY ISSUES

### LEVEL 1 - DIRECT: NSG Blocking
```
# Connection times out
```

**Investigation**:
```bash
# Check NSG rules
az network nsg rule list -g myRG --nsg-name myNSG -o table

# Check effective rules on NIC
az network nic list-effective-nsg -g myRG -n myNIC
```

**Solution**:
```bash
# Add allow rule
az network nsg rule create -g myRG --nsg-name myNSG -n AllowHTTP --priority 100 --direction Inbound --access Allow --protocol Tcp --destination-port-ranges 80
```

---

### LEVEL 2 - INTERMEDIATE: Private Endpoint Not Resolving

**Scenario**: Private endpoint created but DNS doesn't resolve.

**Investigation**:
```bash
# Check private DNS zone
az network private-dns zone show -g myRG -n privatelink.blob.core.windows.net

# Check zone is linked to VNet
az network private-dns link vnet list -g myRG -z privatelink.blob.core.windows.net

# Check A record exists
az network private-dns record-set a list -g myRG -z privatelink.blob.core.windows.net
```

**Common Fixes**:
- Link private DNS zone to VNet
- Create DNS zone group on private endpoint
- Check custom DNS server forwards to Azure DNS (168.63.129.16)

---

### LEVEL 3 - COMPLEX: Intermittent VPN Drops

**Scenario**: Site-to-site VPN works then disconnects.

**Deep Investigation**:
```bash
# Check VPN gateway health
az network vnet-gateway show -g myRG -n myVPNGateway

# Check connections
az network vpn-connection show -g myRG -n myConnection

# Check logs
az monitor activity-log list -g myRG --caller "Microsoft.Network"

# Check IKE logs (via diagnostics)
# Enable in portal: VPN Gateway > Diagnostics settings
```

**Common Causes**:
- MTU issues
- Mismatched policies
- NAT traversal issues
- Overlapping address spaces

---

# SECTION 6: IDENTITY & ACCESS

---

## 6.1 AUTHENTICATION ISSUES

### LEVEL 1 - DIRECT: Managed Identity Not Working
```
Error: DefaultAzureCredential failed to retrieve a token
```

**Cause is VISIBLE**: Managed identity not enabled or no role.

**Solution**:
```bash
# Enable managed identity
az vm identity assign -g myRG -n myVM

# Assign role
az role assignment create --assignee <principal-id> --role "Storage Blob Data Reader" --scope <storage-account-id>
```

---

### LEVEL 1 - DIRECT: Key Vault Access Denied
```
Error: Access denied. Caller was not found on any access policy.
```

**Solution**:
```bash
# With RBAC
az role assignment create --assignee <principal-id> --role "Key Vault Secrets User" --scope /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.KeyVault/vaults/myvault

# With access policy (legacy)
az keyvault set-policy -n myvault --object-id <principal-id> --secret-permissions get list
```

---

# SECTION 7: APP SERVICE

---

## 7.1 DEPLOYMENT ISSUES

### LEVEL 1 - DIRECT: Deployment Failed
```
Error: Failed to deploy web package
```

**Investigation**:
```bash
# Check deployment logs
az webapp log deployment show -g myRG -n myapp

# Check Kudu
# https://myapp.scm.azurewebsites.net

# Stream logs
az webapp log tail -g myRG -n myapp
```

---

### LEVEL 2 - INTERMEDIATE: App Slow or 502 Errors

**Investigation**:
```bash
# Check app status
az webapp show -g myRG -n myapp --query state

# Check metrics
# Portal > App Service > Metrics > Http Server Errors, Response Time

# Check Application Insights
# Look for exceptions and slow dependencies

# Restart app
az webapp restart -g myRG -n myapp
```

---

# QUICK REFERENCE: AZURE ERROR CODES

```
COMMON ERROR CODES:
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ OperationNotAllowed         â”‚ Quota exceeded or not permitted     â”‚
â”‚ ResourceNotFound            â”‚ Resource doesn't exist              â”‚
â”‚ AuthorizationFailed         â”‚ Missing RBAC permission             â”‚
â”‚ InvalidParameter            â”‚ Wrong configuration value           â”‚
â”‚ RequestDisallowedByPolicy   â”‚ Azure Policy blocking               â”‚
â”‚ LinkedAuthorizationFailed   â”‚ Cross-subscription permission       â”‚
â”‚ SkuNotAvailable             â”‚ SKU not available in region         â”‚
â”‚ QuotaExceeded               â”‚ Service quota limit                 â”‚
â”‚ Conflict                    â”‚ Resource state conflict             â”‚
â”‚ BadRequest                  â”‚ Invalid request                     â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

## Diagnostic Commands

```bash
# Check activity log for errors
az monitor activity-log list -g myRG --offset 1h --status Failed

# Check resource health
az resource show -g myRG -n myResource --resource-type <type> --query properties.provisioningState

# Get detailed error for failed deployment
az deployment group show -g myRG -n <deployment-name> --query properties.error
```
