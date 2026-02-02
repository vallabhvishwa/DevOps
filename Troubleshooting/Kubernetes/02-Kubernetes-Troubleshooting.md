# Kubernetes Troubleshooting - Complete Guide
## From Basic to Advanced - Every Scenario

---

# HOW TO USE THIS GUIDE

This guide covers THREE levels:
- **LEVEL 1 - DIRECT**: Cause visible in kubectl output/events
- **LEVEL 2 - INTERMEDIATE**: Needs a few steps to find
- **LEVEL 3 - COMPLEX**: Hidden cause, requires deep investigation

---

# SECTION 1: POD STATUS ISSUES

---

## 1.1 POD PENDING

### LEVEL 1 - DIRECT: Pending with Clear Event

**Scenario**: Pod pending, `kubectl describe` shows exactly why.

```bash
$ kubectl get pod myapp
NAME    READY   STATUS    RESTARTS   AGE
myapp   0/1     Pending   0          5m

$ kubectl describe pod myapp
Events:
  Warning  FailedScheduling  0/3 nodes are available: 
  3 Insufficient cpu.
```

**Cause is VISIBLE**: Not enough CPU on any node.

**Solution**:
```bash
# Check node capacity
kubectl describe nodes | grep -A 10 "Allocated resources"

# Option 1: Reduce pod CPU request
resources:
  requests:
    cpu: "100m"  # Lower request

# Option 2: Add more nodes
# Option 3: Delete other pods to free resources
```

---

### LEVEL 1 - DIRECT: Pending Due to Taint

**Scenario**: Pending because of node taint.

```bash
$ kubectl describe pod myapp
Events:
  Warning  FailedScheduling  0/3 nodes are available:
  3 node(s) had taint {dedicated=gpu}, that the pod didn't tolerate.
```

**Cause is VISIBLE**: Pod doesn't tolerate node taint.

**Solution**:
```yaml
# Add toleration to pod spec
tolerations:
- key: "dedicated"
  operator: "Equal"
  value: "gpu"
  effect: "NoSchedule"
```

---

### LEVEL 1 - DIRECT: Pending Due to PVC

**Scenario**: Pending because volume can't be bound.

```bash
$ kubectl describe pod myapp
Events:
  Warning  FailedScheduling  pod has unbound immediate PersistentVolumeClaims
  
$ kubectl get pvc
NAME       STATUS    VOLUME   CAPACITY
data-pvc   Pending                      # PVC not bound!
```

**Cause is VISIBLE**: PVC is pending.

**Solution**:
```bash
# Check PVC events
kubectl describe pvc data-pvc

# Common fixes:
# 1. Create matching PV
# 2. Fix StorageClass name
# 3. Check storage provisioner is working
```

---

### LEVEL 2 - INTERMEDIATE: Pending, Event Says Node Selector

**Scenario**: Pod pending due to node selector.

```bash
$ kubectl describe pod myapp
Events:
  Warning  FailedScheduling  0/3 nodes are available:
  3 node(s) didn't match Pod's node affinity/selector.
```

**Investigation**:
```bash
# Check pod's node selector
kubectl get pod myapp -o jsonpath='{.spec.nodeSelector}'

# Check node labels
kubectl get nodes --show-labels

# Find mismatch
```

**Solution**:
```bash
# Add label to node
kubectl label node node-1 environment=production

# Or fix pod selector
```

---

### LEVEL 3 - COMPLEX: Pending, No Events At All

**Scenario**: Pod pending, describe shows no events.

```bash
$ kubectl describe pod myapp
# Events section is EMPTY
```

**Hidden Causes**:
```
WHY NO EVENTS:
┌─────────────────────────────────────────────────────────────────┐
│  1. Scheduler not running                                       │
│  2. Wrong schedulerName specified                               │
│  3. Scheduling gates (K8s 1.26+)                                │
│  4. Pod stuck in admission                                      │
└─────────────────────────────────────────────────────────────────┘
```

**Deep Investigation**:
```bash
# Check scheduler
kubectl get pods -n kube-system | grep scheduler
kubectl logs -n kube-system -l component=kube-scheduler

# Check pod's scheduler name
kubectl get pod myapp -o jsonpath='{.spec.schedulerName}'
# If custom scheduler specified but not running = stuck

# Check scheduling gates
kubectl get pod myapp -o jsonpath='{.spec.schedulingGates}'
```

---

## 1.2 POD CRASHLOOPBACKOFF

### LEVEL 1 - DIRECT: CrashLoop with Clear Error in Logs

**Scenario**: Pod keeps crashing, logs show why.

```bash
$ kubectl get pod myapp
NAME    READY   STATUS             RESTARTS   AGE
myapp   0/1     CrashLoopBackOff   5          5m

$ kubectl logs myapp
Error: Cannot connect to database at postgres:5432
Connection refused
```

**Cause is VISIBLE**: Can't connect to database.

**Solution**:
```bash
# Check if postgres service exists
kubectl get svc postgres

# Check if postgres pod is running
kubectl get pod -l app=postgres

# Fix database connection or wait for DB to be ready
```

---

### LEVEL 1 - DIRECT: CrashLoop Due to Config Error

**Scenario**: Application crash due to bad configuration.

```bash
$ kubectl logs myapp
ERROR: Invalid configuration in /app/config.yaml
Line 15: syntax error
```

**Cause is VISIBLE**: Configuration syntax error.

**Solution**:
```bash
# Check ConfigMap
kubectl get configmap myapp-config -o yaml

# Fix the syntax error in ConfigMap
kubectl edit configmap myapp-config

# Restart pod
kubectl delete pod myapp
```

---

### LEVEL 1 - DIRECT: CrashLoop - OOMKilled

**Scenario**: Pod keeps getting killed.

```bash
$ kubectl describe pod myapp
Last State:     Terminated
Reason:         OOMKilled
Exit Code:      137
```

**Cause is VISIBLE**: Out of Memory - container exceeded memory limit.

**Solution**:
```yaml
# Increase memory limit
resources:
  limits:
    memory: "512Mi"  # Increase from 256Mi
  requests:
    memory: "256Mi"
```

---

### LEVEL 2 - INTERMEDIATE: CrashLoop - Exit Code But No Helpful Logs

**Scenario**: Container exits but logs don't show error.

```bash
$ kubectl describe pod myapp
Last State:     Terminated
Reason:         Error
Exit Code:      1

$ kubectl logs myapp --previous
# No obvious error
```

**Investigation**:
```bash
# Exit codes meaning:
# 0 = Success (shouldn't exit for long-running)
# 1 = Generic error
# 137 = SIGKILL (OOM or kill -9)
# 139 = Segfault
# 143 = SIGTERM

# Get more details from events
kubectl get events --field-selector involvedObject.name=myapp

# Check if it's a command issue
kubectl get pod myapp -o jsonpath='{.spec.containers[*].command}'
```

---

### LEVEL 3 - COMPLEX: CrashLoop - Container Can't Start (No Logs)

**Scenario**: Container never starts, logs are empty.

```bash
$ kubectl logs myapp
Error from server: container "myapp" in pod "myapp" is waiting to start

$ kubectl logs myapp --previous
Error from server: previous terminated container not found
```

**Hidden Causes**:
```
WHY NO LOGS:
┌─────────────────────────────────────────────────────────────────┐
│  1. Entrypoint not found                                        │
│  2. Binary not executable                                       │
│  3. Missing library (ldd fails)                                 │
│  4. Wrong architecture (arm vs amd64)                           │
│  5. Permission denied on startup script                         │
└─────────────────────────────────────────────────────────────────┘
```

**Deep Investigation**:
```bash
# Override command to debug
kubectl run debug --image=<same-image> --rm -it --command -- /bin/sh
# If shell works, try running the app command manually

# Check if binary exists
kubectl run debug --image=<same-image> --rm -it --command -- ls -la /app/

# Check library dependencies
kubectl run debug --image=<same-image> --rm -it --command -- ldd /app/myapp
```

---

## 1.3 POD IMAGEPULLBACKOFF

### LEVEL 1 - DIRECT: Image Not Found

**Scenario**: Can't pull image.

```bash
$ kubectl describe pod myapp
Events:
  Warning  Failed   Failed to pull image "myregistry/myapp:v1.0": 
  rpc error: manifest unknown
```

**Cause is VISIBLE**: Image tag doesn't exist.

**Solution**:
```bash
# Check if image exists
docker pull myregistry/myapp:v1.0

# Fix the tag
kubectl set image deployment/myapp myapp=myregistry/myapp:v1.1
```

---

### LEVEL 1 - DIRECT: Authentication Required

**Scenario**: Private registry needs credentials.

```bash
$ kubectl describe pod myapp
Events:
  Warning  Failed   Failed to pull image: 
  unauthorized: authentication required
```

**Cause is VISIBLE**: Missing registry credentials.

**Solution**:
```bash
# Create secret
kubectl create secret docker-registry regcred \
  --docker-server=myregistry.com \
  --docker-username=user \
  --docker-password=pass

# Reference in pod
spec:
  imagePullSecrets:
  - name: regcred
```

---

### LEVEL 2 - INTERMEDIATE: Image Pull Timeout

**Scenario**: Image pull times out.

```bash
$ kubectl describe pod myapp
Events:
  Warning  Failed   context deadline exceeded
```

**Investigation**:
```bash
# Test from node
docker pull myregistry/myapp:v1.0

# Check network to registry
curl -v https://myregistry.com/v2/

# Check if registry is overloaded or rate-limited
```

---

## 1.4 POD RUNNING BUT NOT READY

### LEVEL 1 - DIRECT: Readiness Probe Failing

**Scenario**: Pod running but 0/1 ready.

```bash
$ kubectl get pod myapp
NAME    READY   STATUS    RESTARTS   AGE
myapp   0/1     Running   0          5m

$ kubectl describe pod myapp
Events:
  Warning  Unhealthy  Readiness probe failed: 
  Get "http://10.0.0.5:8080/health": connection refused
```

**Cause is VISIBLE**: Readiness probe failing, connection refused.

**Solution**:
```bash
# Check if app is listening on that port
kubectl exec myapp -- ss -tuln

# App might be listening on localhost only
kubectl exec myapp -- curl http://localhost:8080/health
# If this works but probe fails, app binds to 127.0.0.1

# Fix: Make app bind to 0.0.0.0
```

---

### LEVEL 2 - INTERMEDIATE: Probe Timeout

**Scenario**: Probe times out instead of refusing.

```bash
Events:
  Warning  Unhealthy  Readiness probe failed: 
  Get "http://10.0.0.5:8080/health": context deadline exceeded
```

**Investigation**:
```bash
# Test probe endpoint manually
kubectl exec myapp -- curl -v http://localhost:8080/health

# Check if slow startup
kubectl get pod myapp -o jsonpath='{.spec.containers[*].readinessProbe}'

# Increase timeout or add startup probe
```

---

# SECTION 2: SERVICE & NETWORKING

---

## 2.1 SERVICE NOT ACCESSIBLE

### LEVEL 1 - DIRECT: No Endpoints

**Scenario**: Service exists but no response.

```bash
$ kubectl get svc myapp-service
NAME            TYPE        CLUSTER-IP     PORT(S)
myapp-service   ClusterIP   10.96.100.50   80/TCP

$ kubectl get endpoints myapp-service
NAME            ENDPOINTS
myapp-service   <none>    # NO ENDPOINTS!
```

**Cause is VISIBLE**: Service has no endpoints.

**Solution**:
```bash
# Check selector matches pod labels
kubectl get svc myapp-service -o jsonpath='{.spec.selector}'
# app=myapp

kubectl get pods --show-labels | grep myapp
# Check if label matches

# Fix selector or pod label
```

---

### LEVEL 1 - DIRECT: Service Port Mismatch

**Scenario**: Service port doesn't match container port.

```bash
$ kubectl get svc myapp-service -o yaml
spec:
  ports:
  - port: 80
    targetPort: 8080  # Service forwards 80→8080

$ kubectl exec myapp -- ss -tuln
# Shows listening on 3000, not 8080!
```

**Cause is VISIBLE**: targetPort (8080) doesn't match actual port (3000).

**Solution**:
```bash
# Fix service targetPort
kubectl patch svc myapp-service -p '{"spec":{"ports":[{"port":80,"targetPort":3000}]}}'
```

---

### LEVEL 2 - INTERMEDIATE: Service Works Inside Cluster, Not Outside

**Scenario**: Can curl from pod, not from outside.

```bash
# From inside cluster
kubectl exec otherpod -- curl http://myapp-service
# Works!

# From outside
curl http://<node-ip>:<node-port>
# Connection refused
```

**Investigation**:
```bash
# Check service type
kubectl get svc myapp-service
# If ClusterIP, it's internal only!

# Check NodePort range
kubectl get svc myapp-service -o jsonpath='{.spec.ports[*].nodePort}'

# Check firewall on nodes
iptables -L -n | grep <nodeport>
```

---

### LEVEL 3 - COMPLEX: Service Works From Some Pods, Not Others

**Scenario**: Same service works from pod A, not from pod B.

**Hidden Causes**:
```
WHY SELECTIVE FAILURE:
┌─────────────────────────────────────────────────────────────────┐
│  1. Network Policy blocking                                     │
│  2. Different namespaces (needs FQDN)                           │
│  3. Service mesh (mTLS, authorization)                          │
│  4. kube-proxy issue on specific node                           │
└─────────────────────────────────────────────────────────────────┘
```

**Deep Investigation**:
```bash
# Check namespaces
kubectl get pod pod-a -o jsonpath='{.metadata.namespace}'
kubectl get pod pod-b -o jsonpath='{.metadata.namespace}'

# Check Network Policies
kubectl get networkpolicy -A
kubectl describe networkpolicy <policy> 

# Compare pod labels with policy selectors

# Check from each pod
kubectl exec pod-a -- nslookup myapp-service
kubectl exec pod-b -- nslookup myapp-service
```

---

## 2.2 DNS ISSUES

### LEVEL 1 - DIRECT: DNS Not Resolving

**Scenario**: Can't resolve service name.

```bash
$ kubectl exec myapp -- nslookup myservice
** server can't find myservice: NXDOMAIN
```

**Cause is VISIBLE**: Name doesn't exist.

**Solution**:
```bash
# Check if service exists
kubectl get svc myservice -A

# If in different namespace, use FQDN
kubectl exec myapp -- nslookup myservice.other-namespace.svc.cluster.local
```

---

### LEVEL 2 - INTERMEDIATE: DNS Slow

**Scenario**: DNS works but slow.

```bash
$ kubectl exec myapp -- time nslookup google.com
# Takes 5+ seconds
```

**Investigation**:
```bash
# Check CoreDNS pods
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns

# Check CoreDNS metrics
kubectl top pods -n kube-system -l k8s-app=kube-dns

# Check ndots setting (causes extra queries)
kubectl exec myapp -- cat /etc/resolv.conf
```

---

# SECTION 3: STORAGE ISSUES

---

## 3.1 PVC/PV ISSUES

### LEVEL 1 - DIRECT: PVC Pending, No Matching PV

**Scenario**: PVC won't bind.

```bash
$ kubectl get pvc data-pvc
NAME       STATUS    VOLUME   CAPACITY
data-pvc   Pending

$ kubectl describe pvc data-pvc
Events:
  Warning  ProvisioningFailed  no persistent volumes available 
  for this claim and no storage class is set
```

**Cause is VISIBLE**: No StorageClass and no matching PV.

**Solution**:
```yaml
# Add StorageClass to PVC
spec:
  storageClassName: standard  # Or your cluster's StorageClass
```

---

### LEVEL 1 - DIRECT: Wrong Access Mode

**Scenario**: PVC can't bind due to access mode.

```bash
$ kubectl describe pvc data-pvc
Events:
  Warning  ProvisioningFailed  
  AccessModes requested: ReadWriteMany
  Available: ReadWriteOnce
```

**Cause is VISIBLE**: Requested RWX but only RWO available.

**Solution**:
```yaml
# Use compatible access mode
accessModes:
  - ReadWriteOnce

# Or use storage that supports RWX (NFS, Azure Files)
```

---

### LEVEL 2 - INTERMEDIATE: PVC Bound But Pod Can't Mount

**Scenario**: PVC shows Bound, pod stuck in ContainerCreating.

```bash
$ kubectl get pvc data-pvc
NAME       STATUS   VOLUME
data-pvc   Bound    pv-001

$ kubectl describe pod myapp
Events:
  Warning  FailedMount  Unable to attach or mount volumes:
  timed out waiting for the condition
```

**Investigation**:
```bash
# Check if volume is attached to another node
kubectl get volumeattachment

# Check node affinity (zone mismatch)
kubectl get pv pv-001 -o yaml | grep -A 10 nodeAffinity
kubectl get node <node> -o jsonpath='{.metadata.labels}' | grep zone

# Check CSI driver
kubectl get pods -n kube-system | grep csi
kubectl logs -n kube-system <csi-pod>
```

---

# SECTION 4: DEPLOYMENT & WORKLOADS

---

## 4.1 DEPLOYMENT ISSUES

### LEVEL 1 - DIRECT: Rollout Stuck

**Scenario**: Deployment rollout not progressing.

```bash
$ kubectl rollout status deployment/myapp
Waiting for deployment "myapp" rollout to finish: 1 old replicas pending termination
```

**Investigation**:
```bash
# Check pods
kubectl get pods -l app=myapp

# See what's wrong with new pods
kubectl describe pod <new-pod>

# Check deployment events
kubectl describe deployment myapp
```

---

### LEVEL 1 - DIRECT: Insufficient Quota

**Scenario**: Deployment can't create pods.

```bash
$ kubectl describe deployment myapp
Events:
  Warning  FailedCreate  Error creating: 
  exceeded quota: pods=10, requested: 1, used: 10, limited: 10
```

**Cause is VISIBLE**: Resource quota exceeded.

**Solution**:
```bash
# Check quota
kubectl get resourcequota

# Increase quota or delete unused resources
```

---

## 4.2 CONFIGMAP & SECRETS

### LEVEL 1 - DIRECT: ConfigMap Not Found

**Scenario**: Pod fails because ConfigMap missing.

```bash
$ kubectl describe pod myapp
Events:
  Warning  FailedMount  configmap "myapp-config" not found
```

**Cause is VISIBLE**: ConfigMap doesn't exist.

**Solution**:
```bash
# Create the ConfigMap
kubectl create configmap myapp-config --from-file=config.yaml
```

---

### LEVEL 1 - DIRECT: Secret Not Found

**Scenario**: Pod fails because Secret missing.

```bash
$ kubectl describe pod myapp
Events:
  Warning  FailedMount  secret "myapp-secret" not found
```

**Cause is VISIBLE**: Secret doesn't exist.

**Solution**:
```bash
# Create the Secret
kubectl create secret generic myapp-secret --from-literal=password=mypass
```

---

# QUICK REFERENCE

## Pod Status Meanings

```
Pending       → Waiting to be scheduled (check describe for why)
ContainerCreating → Pulling image or mounting volumes
Running       → Container(s) running (check READY column)
Completed     → Successfully finished (Jobs)
Error         → Container exited with error
CrashLoopBackOff → Keeps crashing, waiting before retry
ImagePullBackOff → Can't pull image
Terminating   → Being deleted
```

## Quick Diagnostic Commands

```bash
# Pod not starting?
kubectl describe pod <pod>        # Check Events section
kubectl logs <pod> --previous     # Previous container logs
kubectl get events --sort-by='.lastTimestamp'

# Service not working?
kubectl get endpoints <svc>       # Has endpoints?
kubectl get svc <svc> -o yaml     # Check selector, ports

# Storage issues?
kubectl get pvc                   # PVC status
kubectl get pv                    # PV status
kubectl describe pvc <pvc>        # Why pending?
kubectl get volumeattachment      # Attachment status

# Networking?
kubectl exec <pod> -- nslookup <service>
kubectl exec <pod> -- curl <service>:<port>
```

## Common Event Messages → Solutions

```
"Insufficient cpu/memory"      → Reduce requests or add nodes
"node(s) had taint"            → Add toleration
"unbound PersistentVolumeClaim"→ Fix PVC/StorageClass
"ImagePullBackOff"             → Check image name, credentials
"CrashLoopBackOff"             → Check logs, fix app error
"Readiness probe failed"       → Fix health endpoint
"FailedMount"                  → Check volume, CSI driver
"no endpoints available"       → Check service selector
```
