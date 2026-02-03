# Scripting & Automation - Complete Guide
## Bash, PowerShell, Common Patterns

---

# 1. BASH SCRIPTING

## Script Structure

```bash
#!/bin/bash
# Description: Script purpose
# Author: Your Name
# Date: 2024-01-15

set -euo pipefail  # Exit on error, undefined vars, pipe failures

# Variables
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
LOG_FILE="/var/log/myscript.log"

# Functions
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

cleanup() {
    log "Cleaning up..."
    # Cleanup code here
}

# Trap for cleanup on exit
trap cleanup EXIT

# Main logic
main() {
    log "Starting script..."
    # Main code here
}

main "$@"
```

## Common Patterns

### Error Handling
```bash
#!/bin/bash
set -euo pipefail

# Exit on any error
set -e

# Exit on undefined variable
set -u

# Exit on pipe failure
set -o pipefail

# Custom error handler
error_handler() {
    echo "Error on line $1"
    exit 1
}
trap 'error_handler $LINENO' ERR

# Check command exists
command -v kubectl >/dev/null 2>&1 || { echo "kubectl required"; exit 1; }
```

### Argument Parsing
```bash
#!/bin/bash

# Default values
ENVIRONMENT="dev"
VERBOSE=false

# Usage
usage() {
    cat <<EOF
Usage: $0 [options]

Options:
    -e, --environment   Environment (dev/staging/prod)
    -v, --verbose       Enable verbose output
    -h, --help          Show this help
EOF
    exit 1
}

# Parse arguments
while [[ $# -gt 0 ]]; do
    case $1 in
        -e|--environment)
            ENVIRONMENT="$2"
            shift 2
            ;;
        -v|--verbose)
            VERBOSE=true
            shift
            ;;
        -h|--help)
            usage
            ;;
        *)
            echo "Unknown option: $1"
            usage
            ;;
    esac
done

echo "Environment: $ENVIRONMENT"
echo "Verbose: $VERBOSE"
```

### File Operations
```bash
#!/bin/bash

# Check if file exists
if [[ -f "/path/to/file" ]]; then
    echo "File exists"
fi

# Check if directory exists
if [[ -d "/path/to/dir" ]]; then
    echo "Directory exists"
fi

# Read file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < "/path/to/file"

# Create temp file
TEMP_FILE=$(mktemp)
trap "rm -f $TEMP_FILE" EXIT

# Find and process files
find /path -name "*.log" -mtime +7 -exec rm {} \;

# Process files safely (handles spaces)
find /path -name "*.txt" -print0 | while IFS= read -r -d '' file; do
    echo "Processing: $file"
done
```

### String Operations
```bash
#!/bin/bash

STRING="hello-world-example"

# Length
echo ${#STRING}  # 19

# Substring
echo ${STRING:0:5}   # hello
echo ${STRING:6}     # world-example

# Replace
echo ${STRING/world/universe}  # hello-universe-example
echo ${STRING//e/E}            # hEllo-world-ExamplE (all)

# Split
IFS='-' read -ra PARTS <<< "$STRING"
for part in "${PARTS[@]}"; do
    echo "$part"
done

# Trim whitespace
STRING="  hello  "
STRING="${STRING#"${STRING%%[![:space:]]*}"}"  # Leading
STRING="${STRING%"${STRING##*[![:space:]]}"}"  # Trailing
```

### Loops and Conditions
```bash
#!/bin/bash

# For loop
for i in {1..5}; do
    echo "Number: $i"
done

# C-style for loop
for ((i=0; i<10; i++)); do
    echo "Index: $i"
done

# While loop
count=0
while [[ $count -lt 5 ]]; do
    echo "Count: $count"
    ((count++))
done

# Loop over array
SERVERS=("server1" "server2" "server3")
for server in "${SERVERS[@]}"; do
    echo "Checking $server"
done

# Conditional
if [[ "$ENV" == "prod" ]]; then
    echo "Production"
elif [[ "$ENV" == "staging" ]]; then
    echo "Staging"
else
    echo "Development"
fi

# Multiple conditions
if [[ -f "$FILE" && -r "$FILE" ]]; then
    echo "File exists and is readable"
fi
```

---

# 2. COMMON AUTOMATION SCRIPTS

## Health Check Script
```bash
#!/bin/bash
set -euo pipefail

SERVICES=("api" "web" "worker")
HEALTHCHECK_PATH="/health"
TIMEOUT=5

check_service() {
    local service=$1
    local url="http://${service}.internal${HEALTHCHECK_PATH}"
    
    if curl -sf --max-time "$TIMEOUT" "$url" > /dev/null; then
        echo "✓ $service is healthy"
        return 0
    else
        echo "✗ $service is unhealthy"
        return 1
    fi
}

main() {
    local failed=0
    
    for service in "${SERVICES[@]}"; do
        if ! check_service "$service"; then
            ((failed++))
        fi
    done
    
    if [[ $failed -gt 0 ]]; then
        echo "Health check failed: $failed service(s) unhealthy"
        exit 1
    fi
    
    echo "All services healthy"
}

main
```

## Log Rotation Script
```bash
#!/bin/bash
set -euo pipefail

LOG_DIR="/var/log/myapp"
MAX_AGE_DAYS=30
MAX_SIZE_MB=100

rotate_logs() {
    # Compress logs older than 1 day
    find "$LOG_DIR" -name "*.log" -mtime +1 -exec gzip {} \;
    
    # Delete compressed logs older than MAX_AGE_DAYS
    find "$LOG_DIR" -name "*.log.gz" -mtime +"$MAX_AGE_DAYS" -delete
    
    # Rotate large active logs
    for log in "$LOG_DIR"/*.log; do
        if [[ -f "$log" ]]; then
            size_mb=$(du -m "$log" | cut -f1)
            if [[ $size_mb -gt $MAX_SIZE_MB ]]; then
                mv "$log" "${log}.$(date +%Y%m%d%H%M%S)"
                touch "$log"
            fi
        fi
    done
}

rotate_logs
echo "Log rotation complete"
```

## Database Backup Script
```bash
#!/bin/bash
set -euo pipefail

# Configuration
DB_HOST="${DB_HOST:-localhost}"
DB_NAME="${DB_NAME:-mydb}"
DB_USER="${DB_USER:-postgres}"
BACKUP_DIR="/backups"
RETENTION_DAYS=7

# Create backup directory
mkdir -p "$BACKUP_DIR"

# Generate backup filename
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="${BACKUP_DIR}/${DB_NAME}_${TIMESTAMP}.sql.gz"

# Create backup
echo "Creating backup: $BACKUP_FILE"
pg_dump -h "$DB_HOST" -U "$DB_USER" "$DB_NAME" | gzip > "$BACKUP_FILE"

# Verify backup
if [[ -s "$BACKUP_FILE" ]]; then
    echo "Backup created successfully: $(ls -lh "$BACKUP_FILE")"
else
    echo "Backup failed!"
    exit 1
fi

# Cleanup old backups
echo "Removing backups older than $RETENTION_DAYS days"
find "$BACKUP_DIR" -name "${DB_NAME}_*.sql.gz" -mtime +"$RETENTION_DAYS" -delete

echo "Backup complete"
```

## Deployment Script
```bash
#!/bin/bash
set -euo pipefail

# Configuration
APP_NAME="myapp"
NAMESPACE="production"
IMAGE_TAG="${1:-latest}"

log() {
    echo "[$(date '+%H:%M:%S')] $1"
}

deploy() {
    log "Deploying $APP_NAME:$IMAGE_TAG to $NAMESPACE"
    
    # Update image
    kubectl set image deployment/"$APP_NAME" \
        "$APP_NAME"="myregistry.azurecr.io/$APP_NAME:$IMAGE_TAG" \
        -n "$NAMESPACE"
    
    # Wait for rollout
    log "Waiting for rollout..."
    if kubectl rollout status deployment/"$APP_NAME" -n "$NAMESPACE" --timeout=300s; then
        log "Deployment successful"
    else
        log "Deployment failed, rolling back"
        kubectl rollout undo deployment/"$APP_NAME" -n "$NAMESPACE"
        exit 1
    fi
}

verify() {
    log "Verifying deployment..."
    
    # Check pods are running
    READY=$(kubectl get deployment "$APP_NAME" -n "$NAMESPACE" -o jsonpath='{.status.readyReplicas}')
    DESIRED=$(kubectl get deployment "$APP_NAME" -n "$NAMESPACE" -o jsonpath='{.spec.replicas}')
    
    if [[ "$READY" == "$DESIRED" ]]; then
        log "All $READY/$DESIRED pods ready"
    else
        log "Only $READY/$DESIRED pods ready"
        exit 1
    fi
}

main() {
    deploy
    verify
    log "Deployment complete"
}

main
```

---

# 3. POWERSHELL FOR AZURE

## Azure Automation

```powershell
# Connect to Azure
Connect-AzAccount

# Set subscription
Set-AzContext -Subscription "My Subscription"

# Get all VMs
$vms = Get-AzVM
foreach ($vm in $vms) {
    Write-Host "VM: $($vm.Name) - Status: $($vm.PowerState)"
}

# Start/Stop VMs by tag
$vmsToStop = Get-AzVM | Where-Object { $_.Tags["Environment"] -eq "Dev" }
foreach ($vm in $vmsToStop) {
    Write-Host "Stopping VM: $($vm.Name)"
    Stop-AzVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -Force
}

# Resize VM
$vm = Get-AzVM -ResourceGroupName "myRG" -Name "myVM"
$vm.HardwareProfile.VmSize = "Standard_D4s_v3"
Update-AzVM -VM $vm -ResourceGroupName "myRG"

# Create snapshot
$disk = Get-AzDisk -ResourceGroupName "myRG" -DiskName "myDisk"
$snapshotConfig = New-AzSnapshotConfig -SourceUri $disk.Id -CreateOption Copy -Location "eastus"
New-AzSnapshot -ResourceGroupName "myRG" -SnapshotName "mySnapshot" -Snapshot $snapshotConfig
```

## Resource Cleanup Script

```powershell
# Delete unused resources

# Find and delete unattached disks
$unusedDisks = Get-AzDisk | Where-Object { $_.DiskState -eq "Unattached" }
foreach ($disk in $unusedDisks) {
    Write-Host "Deleting unattached disk: $($disk.Name)"
    Remove-AzDisk -ResourceGroupName $disk.ResourceGroupName -DiskName $disk.Name -Force
}

# Find and delete unused public IPs
$unusedIPs = Get-AzPublicIpAddress | Where-Object { $_.IpConfiguration -eq $null }
foreach ($ip in $unusedIPs) {
    Write-Host "Deleting unused IP: $($ip.Name)"
    Remove-AzPublicIpAddress -ResourceGroupName $ip.ResourceGroupName -Name $ip.Name -Force
}

# Delete old snapshots
$oldSnapshots = Get-AzSnapshot | Where-Object { 
    $_.TimeCreated -lt (Get-Date).AddDays(-30) 
}
foreach ($snap in $oldSnapshots) {
    Write-Host "Deleting old snapshot: $($snap.Name)"
    Remove-AzSnapshot -ResourceGroupName $snap.ResourceGroupName -SnapshotName $snap.Name -Force
}
```

---

# 4. KUBERNETES AUTOMATION

## Namespace Cleanup
```bash
#!/bin/bash
set -euo pipefail

# Delete pods in error states
kubectl get pods --all-namespaces -o json | \
    jq -r '.items[] | select(.status.phase == "Failed") | 
    "\(.metadata.namespace) \(.metadata.name)"' | \
    while read ns name; do
        kubectl delete pod -n "$ns" "$name"
    done

# Delete completed jobs
kubectl delete jobs --all-namespaces --field-selector status.successful=1

# Delete evicted pods
kubectl get pods --all-namespaces -o json | \
    jq -r '.items[] | select(.status.reason == "Evicted") |
    "\(.metadata.namespace) \(.metadata.name)"' | \
    while read ns name; do
        kubectl delete pod -n "$ns" "$name"
    done
```

## Resource Report
```bash
#!/bin/bash
set -euo pipefail

echo "=== Cluster Resource Usage ==="
echo

echo "=== Node Resources ==="
kubectl top nodes

echo
echo "=== Top Pods by CPU ==="
kubectl top pods --all-namespaces --sort-by=cpu | head -20

echo
echo "=== Top Pods by Memory ==="
kubectl top pods --all-namespaces --sort-by=memory | head -20

echo
echo "=== Pods Without Limits ==="
kubectl get pods --all-namespaces -o json | \
    jq -r '.items[] | select(.spec.containers[].resources.limits == null) |
    "\(.metadata.namespace)/\(.metadata.name)"' | head -20
```

---

# 5. CRON JOBS

## Cron Syntax

```
CRON EXPRESSION:
┌───────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌───────────── day of month (1 - 31)
│ │ │ ┌───────────── month (1 - 12)
│ │ │ │ ┌───────────── day of week (0 - 6)
│ │ │ │ │
* * * * *

EXAMPLES:
┌─────────────────────────────────────────────────────────────────┐
│ */5 * * * *     Every 5 minutes                                │
│ 0 * * * *       Every hour                                     │
│ 0 0 * * *       Every day at midnight                          │
│ 0 0 * * 0       Every Sunday at midnight                       │
│ 0 2 * * *       Every day at 2 AM                              │
│ 0 0 1 * *       First day of every month                       │
│ 0 9-17 * * 1-5  Hourly 9-5, Mon-Fri                            │
└─────────────────────────────────────────────────────────────────┘
```

## Crontab Example

```bash
# Edit crontab
crontab -e

# List crontabs
crontab -l

# Example crontab entries:
# Backup database daily at 2 AM
0 2 * * * /scripts/backup-db.sh >> /var/log/backup.log 2>&1

# Rotate logs every hour
0 * * * * /scripts/rotate-logs.sh

# Health check every 5 minutes
*/5 * * * * /scripts/health-check.sh || /scripts/alert.sh

# Cleanup old files weekly
0 0 * * 0 find /tmp -mtime +7 -delete
```

---

# 6. BEST PRACTICES

```
SCRIPTING BEST PRACTICES:
┌─────────────────────────────────────────────────────────────────┐
│ ✓ Use 'set -euo pipefail' in bash                             │
│ ✓ Always quote variables                                       │
│ ✓ Use functions for reusable code                             │
│ ✓ Add logging with timestamps                                 │
│ ✓ Implement proper error handling                              │
│ ✓ Use meaningful variable names                               │
│ ✓ Add comments and documentation                               │
│ ✓ Test scripts in safe environment first                      │
│ ✓ Use version control for scripts                             │
│ ✓ Implement idempotency (safe to run multiple times)          │
│ ✓ Add cleanup/trap for temporary files                        │
│ ✓ Use configuration files for settings                        │
│ ✓ Log to file, not just stdout                                │
│ ✓ Send alerts on failure                                       │
└─────────────────────────────────────────────────────────────────┘
```
