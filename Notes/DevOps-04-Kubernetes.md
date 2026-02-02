# The Complete DevOps Engineer's Reference Guide
## Part 4: Kubernetes Deep Dive

---

# Chapter 12: Kubernetes Architecture

## 12.1 What is Kubernetes?

Kubernetes (K8s) is an open-source container orchestration platform. It automates deployment, scaling, and management of containerized applications.

### Why Kubernetes?

```
Problems Kubernetes Solves:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  Without Kubernetes:                                                │
│  ├── Manual deployment of containers                                │
│  ├── Manual scaling (add/remove containers)                         │
│  ├── Manual load balancing                                          │
│  ├── Manual health checking and restart                             │
│  ├── Manual rollout/rollback                                        │
│  ├── Managing configuration across servers                          │
│  └── Service discovery (how do services find each other?)           │
│                                                                     │
│  With Kubernetes:                                                   │
│  ├── Declarative deployment ("I want 3 replicas")                   │
│  ├── Automatic scaling (based on CPU, memory, custom metrics)       │
│  ├── Built-in load balancing (Services)                            │
│  ├── Self-healing (restarts failed containers)                      │
│  ├── Rolling updates and rollbacks                                  │
│  ├── ConfigMaps and Secrets for configuration                       │
│  └── Built-in service discovery (DNS)                               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

Kubernetes Philosophy: Declarative Configuration
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  Imperative (what most scripts do):                                 │
│  "Run container A, then run container B, then create load balancer" │
│  - You describe HOW to reach desired state                          │
│                                                                     │
│  Declarative (Kubernetes way):                                      │
│  "I want 3 replicas of container A behind a load balancer"          │
│  - You describe WHAT you want                                       │
│  - Kubernetes figures out how to get there                          │
│  - Kubernetes maintains that state                                  │
│                                                                     │
│  Benefits of Declarative:                                           │
│  ├── Self-healing: If replica dies, K8s creates new one            │
│  ├── Idempotent: Apply same config multiple times = same result     │
│  ├── Version controlled: YAML files in Git                          │
│  └── Auditable: Config shows what's deployed                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 12.2 Kubernetes Architecture

```
Kubernetes Cluster Architecture:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│                        CONTROL PLANE                                │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                                                             │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │   │
│  │  │             │  │             │  │                     │ │   │
│  │  │  API Server │  │    etcd     │  │     Scheduler       │ │   │
│  │  │             │  │             │  │                     │ │   │
│  │  └─────────────┘  └─────────────┘  └─────────────────────┘ │   │
│  │                                                             │   │
│  │  ┌─────────────────────────┐  ┌─────────────────────────┐  │   │
│  │  │                         │  │                         │  │   │
│  │  │   Controller Manager    │  │  Cloud Controller Mgr   │  │   │
│  │  │                         │  │       (AKS/EKS/GKE)     │  │   │
│  │  └─────────────────────────┘  └─────────────────────────┘  │   │
│  │                                                             │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│                              │ kubectl, API calls                   │
│                              ▼                                      │
│                         WORKER NODES                                │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                                                             │   │
│  │   Node 1                    Node 2                   Node N │   │
│  │  ┌──────────────────┐    ┌──────────────────┐             │   │
│  │  │ kubelet          │    │ kubelet          │             │   │
│  │  │ kube-proxy       │    │ kube-proxy       │    ...      │   │
│  │  │ Container Runtime│    │ Container Runtime│             │   │
│  │  │ (containerd)     │    │ (containerd)     │             │   │
│  │  │                  │    │                  │             │   │
│  │  │ ┌─Pod──┐┌─Pod──┐│    │ ┌─Pod──┐┌─Pod──┐│             │   │
│  │  │ │ C1  ││ C2  ││    │ │ C3  ││ C4  ││             │   │
│  │  │ └─────┘└─────┘│    │ └─────┘└─────┘│             │   │
│  │  └──────────────────┘    └──────────────────┘             │   │
│  │                                                             │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Control Plane Components

```
Control Plane Components (Detailed):
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  kube-apiserver                                                     │
│  ├── The central hub of Kubernetes                                  │
│  ├── All communication goes through API server                      │
│  ├── RESTful API for all operations                                 │
│  ├── Handles authentication and authorization                       │
│  ├── Validates and processes API requests                           │
│  ├── Stores data in etcd                                           │
│  └── Only component that directly talks to etcd                    │
│                                                                     │
│  Port: 6443 (HTTPS)                                                 │
│                                                                     │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                     │
│  etcd                                                               │
│  ├── Distributed key-value store                                    │
│  ├── Stores ALL cluster state and configuration                     │
│  ├── Uses Raft consensus algorithm                                  │
│  ├── Single source of truth                                         │
│  ├── Typically runs as 3 or 5 nodes for HA                         │
│  └── CRITICAL: Backup etcd = Backup cluster                        │
│                                                                     │
│  Port: 2379 (client), 2380 (peer)                                   │
│                                                                     │
│  What's stored in etcd:                                             │
│  - All Kubernetes objects (pods, services, deployments, etc.)       │
│  - Cluster configuration                                            │
│  - Secrets (encrypted)                                              │
│  - Service accounts                                                 │
│                                                                     │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                     │
│  kube-scheduler                                                     │
│  ├── Watches for new pods with no assigned node                     │
│  ├── Selects best node for pod placement                           │
│  ├── Considers: resources, affinity, taints, constraints           │
│  ├── Does NOT actually start the pod (kubelet does)                │
│  └── Just assigns pod to node (updates pod spec)                   │
│                                                                     │
│  Scheduling factors:                                                │
│  - Resource requests (CPU, memory)                                  │
│  - Node selectors                                                   │
│  - Affinity/Anti-affinity rules                                     │
│  - Taints and tolerations                                           │
│  - Pod priority                                                     │
│  - Data locality                                                    │
│                                                                     │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                     │
│  kube-controller-manager                                            │
│  ├── Runs controller loops                                          │
│  ├── Controllers watch state and make corrections                   │
│  └── Each controller is a separate loop                            │
│                                                                     │
│  Important controllers:                                             │
│  - Node Controller: Monitors node health                            │
│  - Replication Controller: Maintains replica count                  │
│  - ReplicaSet Controller: Same, but with selectors                 │
│  - Deployment Controller: Manages rollouts                          │
│  - Service Controller: Creates cloud load balancers                │
│  - Endpoint Controller: Populates Endpoints objects                │
│  - ServiceAccount Controller: Creates default ServiceAccounts      │
│  - Namespace Controller: Handles namespace lifecycle               │
│  - Job Controller: Runs pods to completion                         │
│  - CronJob Controller: Creates jobs on schedule                    │
│                                                                     │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                     │
│  cloud-controller-manager                                           │
│  ├── Cloud-specific control logic                                   │
│  ├── Separates cloud code from Kubernetes core                     │
│  └── Runs cloud-specific controllers                               │
│                                                                     │
│  Cloud controllers (AKS example):                                   │
│  - Node Controller: Handles Azure VM lifecycle                      │
│  - Route Controller: Configures Azure routes                       │
│  - Service Controller: Creates Azure Load Balancers                │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Node Components

```
Node Components (Detailed):
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  kubelet                                                            │
│  ├── Primary node agent                                             │
│  ├── Runs on every node                                             │
│  ├── Communicates with API server                                   │
│  ├── Manages pod lifecycle on the node                              │
│  ├── Pulls container images                                         │
│  ├── Starts/stops containers                                        │
│  ├── Reports node and pod status                                    │
│  ├── Runs liveness and readiness probes                            │
│  └── Mounts volumes                                                 │
│                                                                     │
│  Port: 10250 (kubelet API)                                          │
│                                                                     │
│  kubelet does NOT manage containers not created by Kubernetes       │
│                                                                     │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                     │
│  kube-proxy                                                         │
│  ├── Network proxy on each node                                     │
│  ├── Implements Service abstraction                                 │
│  ├── Maintains network rules for service traffic                    │
│  ├── Load balances traffic to pod endpoints                        │
│  └── Uses iptables or IPVS mode                                    │
│                                                                     │
│  How it works:                                                      │
│  1. Watches Services and Endpoints                                  │
│  2. Creates iptables/IPVS rules                                    │
│  3. Rules redirect service IP to pod IPs                           │
│                                                                     │
│  Example: Service ClusterIP 10.96.0.10 → Pod IPs 10.244.1.5,       │
│           10.244.2.7, 10.244.3.3 (round-robin)                     │
│                                                                     │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                     │
│  Container Runtime                                                  │
│  ├── Software that runs containers                                  │
│  ├── Implements Container Runtime Interface (CRI)                   │
│  └── Common options:                                                │
│      - containerd (default, most common)                           │
│      - CRI-O (RHEL/OpenShift)                                      │
│      - Docker (deprecated in K8s, but containerd works)            │
│                                                                     │
│  containerd:                                                        │
│  - Industry-standard container runtime                              │
│  - Manages complete container lifecycle                             │
│  - Pulls images, manages storage, runs containers                   │
│  - Extracted from Docker, now standalone                            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 12.3 The API Server and API Objects

```
Kubernetes API:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  The API Server exposes a RESTful API:                              │
│                                                                     │
│  Base URL: https://<api-server>:6443                               │
│                                                                     │
│  API Groups:                                                        │
│  ├── Core (legacy): /api/v1                                        │
│  │   - Pods, Services, ConfigMaps, Secrets, Namespaces             │
│  │   - PersistentVolumes, PersistentVolumeClaims                   │
│  │   - ServiceAccounts, Nodes, Events                              │
│  │                                                                  │
│  ├── Apps: /apis/apps/v1                                           │
│  │   - Deployments, DaemonSets, StatefulSets, ReplicaSets          │
│  │                                                                  │
│  ├── Batch: /apis/batch/v1                                         │
│  │   - Jobs, CronJobs                                              │
│  │                                                                  │
│  ├── Networking: /apis/networking.k8s.io/v1                        │
│  │   - Ingress, NetworkPolicy, IngressClass                        │
│  │                                                                  │
│  ├── RBAC: /apis/rbac.authorization.k8s.io/v1                      │
│  │   - Roles, ClusterRoles, RoleBindings, ClusterRoleBindings      │
│  │                                                                  │
│  ├── Storage: /apis/storage.k8s.io/v1                              │
│  │   - StorageClasses, VolumeAttachments                           │
│  │                                                                  │
│  └── Many more...                                                   │
│                                                                     │
│  Example API paths:                                                 │
│  GET  /api/v1/namespaces/default/pods                              │
│  GET  /api/v1/namespaces/default/pods/my-pod                       │
│  POST /api/v1/namespaces/default/pods                              │
│  PUT  /api/v1/namespaces/default/pods/my-pod                       │
│  DELETE /api/v1/namespaces/default/pods/my-pod                     │
│                                                                     │
│  List API resources:                                                │
│  kubectl api-resources                                              │
│  kubectl api-versions                                               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

# Chapter 13: Kubernetes Objects In Depth

## 13.1 Pods

The Pod is the smallest deployable unit in Kubernetes. It's a group of one or more containers that share storage, network, and lifecycle.

```yaml
# Complete Pod specification with all common fields
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  namespace: default                  # Namespace (default if not specified)
  labels:                             # Key-value pairs for selection
    app: myapp
    environment: production
    tier: frontend
  annotations:                        # Metadata not used for selection
    description: "My application pod"
    prometheus.io/scrape: "true"
    prometheus.io/port: "8080"
spec:
  # Node selection
  nodeSelector:                       # Simple node selection
    disktype: ssd
  
  # Affinity rules (advanced scheduling)
  affinity:
    nodeAffinity:                     # Which nodes to schedule on
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: topology.kubernetes.io/zone
            operator: In
            values:
            - zone-a
            - zone-b
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        preference:
          matchExpressions:
          - key: node-type
            operator: In
            values:
            - high-memory
    
    podAffinity:                      # Schedule near other pods
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: cache
        topologyKey: kubernetes.io/hostname
    
    podAntiAffinity:                  # Schedule away from other pods
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchLabels:
              app: myapp
          topologyKey: kubernetes.io/hostname
  
  # Tolerations (allow scheduling on tainted nodes)
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "database"
    effect: "NoSchedule"
  - key: "node.kubernetes.io/not-ready"
    operator: "Exists"
    effect: "NoExecute"
    tolerationSeconds: 300
  
  # Service Account
  serviceAccountName: my-service-account
  automountServiceAccountToken: true
  
  # Security context (pod level)
  securityContext:
    runAsUser: 1000
    runAsGroup: 1000
    fsGroup: 1000
    runAsNonRoot: true
    seccompProfile:
      type: RuntimeDefault
  
  # DNS configuration
  dnsPolicy: ClusterFirst              # Default, use cluster DNS
  # dnsPolicy: Default                 # Use node's DNS
  # dnsPolicy: None                    # Use dnsConfig below
  dnsConfig:
    nameservers:
    - 8.8.8.8
    searches:
    - my-namespace.svc.cluster.local
    options:
    - name: ndots
      value: "5"
  
  # Host settings
  hostname: my-pod-hostname
  subdomain: my-subdomain
  hostNetwork: false                  # Use host networking
  hostPID: false                      # Use host PID namespace
  hostIPC: false                      # Use host IPC namespace
  
  # Restart policy
  restartPolicy: Always               # Always, OnFailure, Never
  
  # Termination
  terminationGracePeriodSeconds: 30
  
  # Init containers (run before main containers)
  initContainers:
  - name: init-database
    image: busybox:1.35
    command: ['sh', '-c', 'until nc -z database:5432; do sleep 2; done']
  
  # Main containers
  containers:
  - name: myapp
    image: myapp:1.0.0
    imagePullPolicy: IfNotPresent     # Always, IfNotPresent, Never
    
    # Command and arguments
    command: ["/bin/sh"]              # Overrides ENTRYPOINT
    args: ["-c", "echo hello"]        # Overrides CMD
    
    # Working directory
    workingDir: /app
    
    # Ports
    ports:
    - name: http
      containerPort: 8080
      protocol: TCP
    - name: metrics
      containerPort: 9090
      protocol: TCP
    
    # Environment variables
    env:
    - name: SIMPLE_VAR
      value: "simple-value"
    - name: POD_NAME                  # From downward API
      valueFrom:
        fieldRef:
          fieldPath: metadata.name
    - name: POD_IP
      valueFrom:
        fieldRef:
          fieldPath: status.podIP
    - name: NODE_NAME
      valueFrom:
        fieldRef:
          fieldPath: spec.nodeName
    - name: CPU_LIMIT
      valueFrom:
        resourceFieldRef:
          containerName: myapp
          resource: limits.cpu
    - name: CONFIG_VALUE              # From ConfigMap
      valueFrom:
        configMapKeyRef:
          name: my-config
          key: config-key
    - name: SECRET_VALUE              # From Secret
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: secret-key
    
    # Environment from ConfigMap/Secret
    envFrom:
    - configMapRef:
        name: my-config
    - secretRef:
        name: my-secret
    
    # Resource requests and limits
    resources:
      requests:                       # Scheduling guarantee
        memory: "256Mi"
        cpu: "250m"                   # 0.25 CPU cores
      limits:                         # Maximum allowed
        memory: "512Mi"
        cpu: "500m"                   # 0.5 CPU cores
    
    # Volume mounts
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
      readOnly: true
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
    - name: data-volume
      mountPath: /data
    - name: temp-volume
      mountPath: /tmp
    
    # Liveness probe (is container alive?)
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 30         # Wait before first probe
      periodSeconds: 10               # Probe every N seconds
      timeoutSeconds: 5               # Probe timeout
      successThreshold: 1             # Required successes
      failureThreshold: 3             # Failures before restart
    
    # Readiness probe (is container ready for traffic?)
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
      timeoutSeconds: 3
      successThreshold: 1
      failureThreshold: 3
    
    # Startup probe (for slow-starting containers)
    startupProbe:
      httpGet:
        path: /startup
        port: 8080
      initialDelaySeconds: 0
      periodSeconds: 10
      timeoutSeconds: 5
      successThreshold: 1
      failureThreshold: 30            # 30 * 10s = 5 minutes to start
    
    # Container security context
    securityContext:
      runAsUser: 1000
      runAsGroup: 1000
      runAsNonRoot: true
      readOnlyRootFilesystem: true
      allowPrivilegeEscalation: false
      capabilities:
        drop:
        - ALL
        add:
        - NET_BIND_SERVICE
    
    # Lifecycle hooks
    lifecycle:
      postStart:
        exec:
          command: ["/bin/sh", "-c", "echo Container started"]
      preStop:
        exec:
          command: ["/bin/sh", "-c", "sleep 15"]
  
  # Volumes
  volumes:
  - name: config-volume
    configMap:
      name: my-config
      items:
      - key: config.yaml
        path: config.yaml
        mode: 0644
  
  - name: secret-volume
    secret:
      secretName: my-secret
      items:
      - key: password
        path: password.txt
        mode: 0400
  
  - name: data-volume
    persistentVolumeClaim:
      claimName: my-pvc
  
  - name: temp-volume
    emptyDir:
      sizeLimit: 500Mi
  
  # Image pull secrets
  imagePullSecrets:
  - name: my-registry-secret
```

### Understanding Pod Phases and Conditions

```
Pod Phases:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  Pending                                                            │
│  ├── Pod accepted but not yet scheduled                            │
│  ├── Waiting for scheduler to assign node                          │
│  ├── Waiting for image pull                                        │
│  └── Waiting for volume mount                                      │
│                                                                     │
│  Running                                                            │
│  ├── Pod bound to node                                              │
│  ├── All containers created                                         │
│  └── At least one container running or starting                    │
│                                                                     │
│  Succeeded                                                          │
│  ├── All containers terminated successfully                         │
│  └── Will not restart (used for Jobs)                              │
│                                                                     │
│  Failed                                                             │
│  ├── All containers terminated                                      │
│  └── At least one exited with error                                │
│                                                                     │
│  Unknown                                                            │
│  └── Node communication failure                                     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

Container States:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  Waiting                                                            │
│  ├── Not running yet                                                │
│  └── Reasons: ContainerCreating, ImagePullBackOff,                 │
│       CrashLoopBackOff, ErrImagePull                               │
│                                                                     │
│  Running                                                            │
│  └── Container executing                                            │
│                                                                     │
│  Terminated                                                         │
│  └── Container finished execution                                   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

Common Conditions:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  PodScheduled      : Pod has been scheduled to a node              │
│  Initialized       : Init containers completed                      │
│  ContainersReady   : All containers ready                          │
│  Ready             : Pod ready to serve requests                    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 13.2 Controllers

Controllers manage Pods. You rarely create Pods directly; instead, you create controllers.

### Deployment

Deployments manage stateless applications with rolling updates.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  labels:
    app: myapp
spec:
  replicas: 3                         # Desired number of pods
  
  # How to identify pods this deployment manages
  selector:
    matchLabels:
      app: myapp
    # matchExpressions:               # Alternative/additional
    # - key: environment
    #   operator: In
    #   values: [production, staging]
  
  # Update strategy
  strategy:
    type: RollingUpdate               # or Recreate
    rollingUpdate:
      maxSurge: 1                     # Max pods over desired (1 or 25%)
      maxUnavailable: 0               # Max pods unavailable (0 or 25%)
  
  # Revision history
  revisionHistoryLimit: 10            # Keep last 10 ReplicaSets
  
  # Progress deadline
  progressDeadlineSeconds: 600        # Fail if no progress in 10 min
  
  # Minimum ready time
  minReadySeconds: 10                 # Wait 10s before considering ready
  
  # Pause rollout
  paused: false
  
  # Pod template (same as Pod spec)
  template:
    metadata:
      labels:
        app: myapp                    # Must match selector
      annotations:
        prometheus.io/scrape: "true"
    spec:
      containers:
      - name: myapp
        image: myapp:1.0.0
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

```bash
# Deployment operations
kubectl apply -f deployment.yaml      # Create/update
kubectl get deployments               # List deployments
kubectl describe deployment my-deployment
kubectl get rs                        # List ReplicaSets
kubectl get pods -l app=myapp         # List pods

# Scaling
kubectl scale deployment my-deployment --replicas=5
kubectl autoscale deployment my-deployment --min=3 --max=10 --cpu-percent=80

# Rolling update
kubectl set image deployment/my-deployment myapp=myapp:2.0.0
kubectl rollout status deployment/my-deployment
kubectl rollout history deployment/my-deployment
kubectl rollout undo deployment/my-deployment
kubectl rollout undo deployment/my-deployment --to-revision=2
kubectl rollout restart deployment/my-deployment
kubectl rollout pause deployment/my-deployment
kubectl rollout resume deployment/my-deployment
```

### StatefulSet

StatefulSets are for stateful applications that need stable identities.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql                  # Headless service name
  replicas: 3
  
  selector:
    matchLabels:
      app: mysql
  
  # Pod management policy
  podManagementPolicy: OrderedReady   # or Parallel
  
  # Update strategy
  updateStrategy:
    type: RollingUpdate               # or OnDelete
    rollingUpdate:
      partition: 0                    # Don't update pods with ordinal < N
  
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
          name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: root-password
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
  
  # Volume claim template (creates PVC for each pod)
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: premium-ssd
      resources:
        requests:
          storage: 10Gi
---
# Headless service for StatefulSet
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  clusterIP: None                     # Headless service
  selector:
    app: mysql
  ports:
  - port: 3306
    name: mysql
```

```
StatefulSet Guarantees:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  1. Stable Network Identity                                         │
│     Pod names: mysql-0, mysql-1, mysql-2 (ordinal index)           │
│     DNS: mysql-0.mysql.namespace.svc.cluster.local                 │
│                                                                     │
│  2. Stable Storage                                                  │
│     Each pod gets its own PVC                                       │
│     PVC persists even if pod is deleted                            │
│     Same PVC reattached when pod recreated                         │
│                                                                     │
│  3. Ordered Deployment and Scaling                                  │
│     Created in order: 0, 1, 2, ...                                 │
│     Deleted in reverse: ..., 2, 1, 0                               │
│     Pod N+1 not created until Pod N is ready                       │
│                                                                     │
│  4. Ordered Updates                                                 │
│     Updated in reverse order (highest to lowest)                   │
│     Each pod updated only after previous is ready                  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### DaemonSet

DaemonSets run one pod per node (e.g., log collectors, monitoring agents).

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  labels:
    app: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  
  # Update strategy
  updateStrategy:
    type: RollingUpdate               # or OnDelete
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 0
  
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      # Run on all nodes including masters
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      - key: node-role.kubernetes.io/control-plane
        effect: NoSchedule
      
      containers:
      - name: fluentd
        image: fluent/fluentd:v1.14
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: containers
          mountPath: /var/lib/docker/containers
          readOnly: true
      
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: containers
        hostPath:
          path: /var/lib/docker/containers
```

### Job and CronJob

```yaml
# Job - run to completion
apiVersion: batch/v1
kind: Job
metadata:
  name: backup-job
spec:
  completions: 1                      # How many pods should complete
  parallelism: 1                      # How many pods run in parallel
  backoffLimit: 4                     # Retries before marking failed
  activeDeadlineSeconds: 600          # Max time for job
  ttlSecondsAfterFinished: 100        # Clean up after completion
  
  template:
    spec:
      restartPolicy: Never            # or OnFailure
      containers:
      - name: backup
        image: backup-tool:latest
        command: ["/backup.sh"]
        volumeMounts:
        - name: data
          mountPath: /data
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: data-pvc
---
# CronJob - scheduled job
apiVersion: batch/v1
kind: CronJob
metadata:
  name: daily-backup
spec:
  schedule: "0 2 * * *"               # 2 AM daily (cron format)
  timeZone: "America/New_York"        # Optional timezone
  
  concurrencyPolicy: Forbid           # Allow, Forbid, Replace
  startingDeadlineSeconds: 200        # Miss deadline = skip
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  suspend: false
  
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: backup
            image: backup-tool:latest
            command: ["/backup.sh"]
```

## 13.3 Services

Services provide stable networking for pods.

```yaml
# ClusterIP (default) - Internal only
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ClusterIP                     # Default
  selector:
    app: myapp
  ports:
  - name: http
    port: 80                          # Service port
    targetPort: 8080                  # Container port
    protocol: TCP
  # clusterIP: 10.96.100.50          # Specific IP (optional)
  # clusterIP: None                   # Headless service
---
# NodePort - Expose on node's IP
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080                   # 30000-32767, or auto-assigned
---
# LoadBalancer - Cloud load balancer
apiVersion: v1
kind: Service
metadata:
  name: my-loadbalancer
  annotations:
    # Azure-specific annotations
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
  # loadBalancerIP: 10.0.0.100       # Request specific IP
  # loadBalancerSourceRanges:        # Firewall source ranges
  # - 10.0.0.0/8
---
# ExternalName - DNS alias to external service
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: database.example.com
```

```
Service Types Explained:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  ClusterIP (default)                                                │
│  ├── Internal cluster IP                                            │
│  ├── Only accessible within cluster                                 │
│  └── Use case: Internal services, databases                        │
│                                                                     │
│  NodePort                                                           │
│  ├── Exposes on each node's IP at static port                      │
│  ├── Accessible from outside cluster                                │
│  ├── Port range: 30000-32767                                       │
│  └── Use case: Development, on-prem without LB                     │
│                                                                     │
│  LoadBalancer                                                       │
│  ├── Creates external load balancer                                 │
│  ├── Cloud-provider specific                                        │
│  └── Use case: Production external access                          │
│                                                                     │
│  ExternalName                                                       │
│  ├── DNS CNAME to external service                                  │
│  ├── No proxying, just DNS                                          │
│  └── Use case: External databases, APIs                            │
│                                                                     │
│  Headless (ClusterIP: None)                                         │
│  ├── No cluster IP                                                  │
│  ├── DNS returns pod IPs directly                                   │
│  └── Use case: StatefulSets, peer discovery                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 13.4 Ingress

Ingress provides HTTP/HTTPS routing to Services.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    # NGINX Ingress Controller annotations
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "50m"
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "60"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx             # Ingress controller to use
  
  # TLS configuration
  tls:
  - hosts:
    - myapp.example.com
    - api.example.com
    secretName: tls-secret            # Contains tls.crt and tls.key
  
  # Routing rules
  rules:
  - host: myapp.example.com           # Host header matching
    http:
      paths:
      - path: /
        pathType: Prefix              # Prefix, Exact, ImplementationSpecific
        backend:
          service:
            name: frontend
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api
            port:
              number: 80
  
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api
            port:
              number: 80
  
  # Default backend (when no rules match)
  defaultBackend:
    service:
      name: default-backend
      port:
        number: 80
```

## 13.5 ConfigMaps and Secrets

```yaml
# ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  # Simple key-value
  database_host: "mysql.default.svc.cluster.local"
  log_level: "info"
  
  # Multi-line file
  config.yaml: |
    server:
      port: 8080
      timeout: 30s
    database:
      host: mysql
      port: 3306
    logging:
      level: info
      format: json
  
  # Properties file
  app.properties: |
    app.name=myapp
    app.version=1.0.0
---
# Secret
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque                          # Generic secret
data:
  # Values must be base64 encoded
  username: YWRtaW4=                  # echo -n 'admin' | base64
  password: cGFzc3dvcmQxMjM=          # echo -n 'password123' | base64
stringData:
  # Or use stringData (not encoded)
  api-key: "my-api-key-value"
---
# TLS Secret
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: <base64-encoded-cert>
  tls.key: <base64-encoded-key>
---
# Docker registry secret
apiVersion: v1
kind: Secret
metadata:
  name: regcred
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <base64-encoded-docker-config>

# Create with kubectl:
# kubectl create secret docker-registry regcred \
#   --docker-server=myregistry.azurecr.io \
#   --docker-username=username \
#   --docker-password=password
```

## 13.6 Storage

```yaml
# StorageClass
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: premium-ssd
provisioner: disk.csi.azure.com       # Azure Disk CSI
parameters:
  skuName: Premium_LRS
  cachingmode: ReadOnly
reclaimPolicy: Retain                 # Delete or Retain
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
---
# PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce                     # RWO, ROX, RWX, RWOP
  storageClassName: premium-ssd
  resources:
    requests:
      storage: 10Gi
---
# PersistentVolume (manual, for pre-existing storage)
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: premium-ssd
  azureDisk:
    diskName: my-azure-disk
    diskURI: /subscriptions/.../resourceGroups/.../providers/Microsoft.Compute/disks/my-disk
    kind: Managed
```

```
Access Modes:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  ReadWriteOnce (RWO)                                               │
│  ├── Mount as read-write by single node                            │
│  └── Multiple pods on SAME node can access                         │
│                                                                     │
│  ReadOnlyMany (ROX)                                                │
│  └── Mount as read-only by many nodes                              │
│                                                                     │
│  ReadWriteMany (RWX)                                               │
│  ├── Mount as read-write by many nodes                             │
│  └── Requires specific storage (NFS, Azure Files)                  │
│                                                                     │
│  ReadWriteOncePod (RWOP)                                           │
│  ├── Mount as read-write by single pod                             │
│  └── Kubernetes 1.22+                                              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 13.7 RBAC (Role-Based Access Control)

```yaml
# ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app
  namespace: default
---
# Role (namespace-scoped)
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]                     # Core API group
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["pods/exec"]
  verbs: ["create"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list"]
---
# RoleBinding (binds Role to subjects)
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: ServiceAccount
  name: my-app
  namespace: default
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
- kind: Group
  name: developers
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
---
# ClusterRole (cluster-scoped)
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["namespaces"]
  verbs: ["get", "list"]
---
# ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-nodes
subjects:
- kind: ServiceAccount
  name: my-app
  namespace: default
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

```
RBAC Verbs:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  Verb      │ Description                                           │
│  ─────────────────────────────────────────────────────────────────  │
│  get       │ Read a single resource                                │
│  list      │ List resources                                        │
│  watch     │ Watch for changes                                     │
│  create    │ Create resources                                      │
│  update    │ Update existing resources                             │
│  patch     │ Partially update resources                            │
│  delete    │ Delete resources                                      │
│  deletecollection │ Delete multiple resources                      │
│                                                                     │
│  Special verbs for specific resources:                              │
│  - pods/exec: create (to exec into pod)                            │
│  - pods/log: get (to read logs)                                    │
│  - pods/portforward: create                                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

# Chapter 14: kubectl Mastery

## 14.1 kubectl Configuration

```bash
# Kubeconfig file location
~/.kube/config                        # Default location
$KUBECONFIG                           # Environment variable

# Kubeconfig structure:
# clusters: API server endpoints
# users: Authentication credentials
# contexts: Combine cluster + user + namespace

# View config
kubectl config view
kubectl config view --minify          # Current context only

# Contexts
kubectl config get-contexts           # List contexts
kubectl config current-context        # Show current
kubectl config use-context <context>  # Switch context
kubectl config set-context --current --namespace=<ns>  # Set default namespace

# Quick switching (install kubectx)
kubectx                               # List contexts
kubectx <context>                     # Switch
kubens                                # List namespaces
kubens <namespace>                    # Switch namespace
```

## 14.2 Essential kubectl Commands

```bash
# Get resources
kubectl get pods                      # List pods
kubectl get pods -o wide              # More details
kubectl get pods -o yaml              # YAML output
kubectl get pods -o json              # JSON output
kubectl get pods --show-labels        # Show labels
kubectl get pods -l app=nginx         # Filter by label
kubectl get pods --field-selector=status.phase=Running
kubectl get pods --sort-by=.metadata.creationTimestamp
kubectl get all                       # All resource types
kubectl get all -A                    # All namespaces

# Custom output
kubectl get pods -o custom-columns='NAME:.metadata.name,STATUS:.status.phase'
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.phase}{"\n"}{end}'

# Describe (detailed info)
kubectl describe pod <pod>
kubectl describe node <node>
kubectl describe deployment <deploy>

# Create/Apply/Delete
kubectl apply -f manifest.yaml        # Create or update
kubectl apply -f directory/           # Apply all in directory
kubectl apply -f https://url/manifest.yaml
kubectl create -f manifest.yaml       # Create only
kubectl delete -f manifest.yaml
kubectl delete pod <pod>
kubectl delete pod <pod> --grace-period=0 --force  # Force delete

# Edit resources
kubectl edit deployment <deploy>      # Opens in editor
kubectl patch deployment <deploy> -p '{"spec":{"replicas":5}}'

# Logs
kubectl logs <pod>                    # Container logs
kubectl logs <pod> -c <container>     # Specific container
kubectl logs <pod> --previous         # Previous container
kubectl logs <pod> -f                 # Follow
kubectl logs <pod> --since=1h         # Last hour
kubectl logs <pod> --tail=100         # Last 100 lines
kubectl logs -l app=nginx             # By label
kubectl logs -l app=nginx --all-containers

# Exec
kubectl exec <pod> -- ls              # Run command
kubectl exec -it <pod> -- /bin/bash   # Interactive shell
kubectl exec -it <pod> -c <container> -- /bin/bash

# Port forwarding
kubectl port-forward <pod> 8080:80    # Pod
kubectl port-forward svc/<svc> 8080:80 # Service
kubectl port-forward deploy/<deploy> 8080:80

# Copy files
kubectl cp <pod>:/path/file ./local-file
kubectl cp ./local-file <pod>:/path/file

# Resource usage
kubectl top pods                      # Pod CPU/memory
kubectl top pods --containers         # Per container
kubectl top nodes                     # Node CPU/memory

# Events
kubectl get events --sort-by='.lastTimestamp'
kubectl get events -A                 # All namespaces
kubectl get events --field-selector type=Warning

# Run temporary pods
kubectl run debug --rm -it --image=busybox -- sh
kubectl run curl --rm -it --image=curlimages/curl -- curl http://service
kubectl run netshoot --rm -it --image=nicolaka/netshoot -- bash

# Debugging
kubectl debug <pod> -it --image=busybox
kubectl debug node/<node> -it --image=ubuntu

# Dry run
kubectl apply -f file.yaml --dry-run=client
kubectl apply -f file.yaml --dry-run=server
kubectl create deployment test --image=nginx --dry-run=client -o yaml

# Diff
kubectl diff -f file.yaml            # Show what would change

# Rollouts
kubectl rollout status deployment/<deploy>
kubectl rollout history deployment/<deploy>
kubectl rollout undo deployment/<deploy>
kubectl rollout restart deployment/<deploy>

# Scale
kubectl scale deployment/<deploy> --replicas=5

# Labels and annotations
kubectl label pods <pod> env=prod
kubectl label pods <pod> env-               # Remove label
kubectl annotate pods <pod> description="My pod"

# Taints and tolerations
kubectl taint nodes <node> key=value:NoSchedule
kubectl taint nodes <node> key:NoSchedule-  # Remove taint

# Cordon/Drain
kubectl cordon <node>                 # Mark unschedulable
kubectl uncordon <node>               # Mark schedulable
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data

# API resources
kubectl api-resources                 # List all resource types
kubectl api-versions                  # List API versions
kubectl explain pod                   # Documentation
kubectl explain pod.spec.containers   # Nested fields
kubectl explain pod --recursive       # All fields
```

---

This concludes Part 4 covering Kubernetes in depth.

**Continue to Part 5 for:**
- AKS (Azure Kubernetes Service) Complete Guide
- Azure Services for DevOps Engineers
- Azure CLI, Resource Management
