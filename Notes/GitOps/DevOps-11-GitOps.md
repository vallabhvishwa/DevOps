# DevOps Engineer's Complete Reference Guide
# Part 11: GitOps with ArgoCD and Flux

---

## Table of Contents

1. [Introduction to GitOps](#1-introduction-to-gitops)
2. [GitOps Principles](#2-gitops-principles)
3. [ArgoCD Deep Dive](#3-argocd-deep-dive)
4. [Flux CD Deep Dive](#4-flux-cd-deep-dive)
5. [Repository Strategies](#5-repository-strategies)
6. [Secret Management](#6-secret-management)
7. [Progressive Delivery](#7-progressive-delivery)
8. [Multi-Cluster GitOps](#8-multi-cluster-gitops)
9. [GitOps Best Practices](#9-gitops-best-practices)

---

## 1. Introduction to GitOps

### 1.1 What is GitOps?

```
GitOps Definition:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  GitOps is an operational framework that uses Git as the        │
│  single source of truth for declarative infrastructure and     │
│  applications.                                                  │
│                                                                 │
│  Traditional CI/CD:                                             │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐      │
│  │   Git   │───►│   CI    │───►│   CD    │───►│ Cluster │      │
│  │  (Code) │    │ (Build) │    │ (Push)  │    │         │      │
│  └─────────┘    └─────────┘    └─────────┘    └─────────┘      │
│                                     │                           │
│                              Push-based ▼                       │
│                                                                 │
│  GitOps:                                                        │
│  ┌─────────┐    ┌─────────┐         ┌─────────┐                │
│  │   Git   │◄──►│ GitOps  │────────►│ Cluster │                │
│  │ (Truth) │    │ Operator│ Sync    │         │                │
│  └─────────┘    └─────────┘◄────────└─────────┘                │
│       ▲              │         Compare                          │
│       │              │                                          │
│       │        Pull-based                                       │
│       │                                                         │
│  Developer pushes changes to Git                                │
│  Operator pulls and applies to cluster                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Benefits of GitOps

```
GitOps Benefits:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. GIT AS SINGLE SOURCE OF TRUTH                               │
│     ├── Version controlled infrastructure                       │
│     ├── Complete audit trail                                    │
│     ├── Easy rollback (git revert)                              │
│     └── Pull request workflow for changes                       │
│                                                                 │
│  2. IMPROVED SECURITY                                           │
│     ├── No direct cluster access needed                         │
│     ├── Credentials stay in cluster                             │
│     ├── Pull-based (no exposed APIs)                            │
│     └── Separation of concerns                                  │
│                                                                 │
│  3. INCREASED RELIABILITY                                       │
│     ├── Automatic drift detection                               │
│     ├── Self-healing (continuous reconciliation)                │
│     ├── Declarative desired state                               │
│     └── Reproducible deployments                                │
│                                                                 │
│  4. FASTER RECOVERY                                             │
│     ├── Disaster recovery: recreate from Git                    │
│     ├── Rollback: git revert, automatic sync                    │
│     └── New clusters: point to Git, done                        │
│                                                                 │
│  5. DEVELOPER EXPERIENCE                                        │
│     ├── Familiar Git workflow                                   │
│     ├── Self-service deployments                                │
│     └── Clear visibility into state                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. GitOps Principles

### 2.1 Core Principles

```
Four Principles of GitOps:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. DECLARATIVE                                                 │
│     ├── Entire system described declaratively                   │
│     ├── Kubernetes YAML, Helm, Kustomize                        │
│     └── What, not how                                           │
│                                                                 │
│  2. VERSIONED AND IMMUTABLE                                     │
│     ├── Desired state stored in Git                             │
│     ├── Version history maintained                              │
│     ├── Immutable versions (tags)                               │
│     └── Source of truth is Git, not cluster                     │
│                                                                 │
│  3. PULLED AUTOMATICALLY                                        │
│     ├── Operators pull from Git (not push)                      │
│     ├── Continuous reconciliation                               │
│     ├── No external access to cluster needed                    │
│     └── Changes detected automatically                          │
│                                                                 │
│  4. CONTINUOUSLY RECONCILED                                     │
│     ├── Agents compare desired vs actual                        │
│     ├── Drift corrected automatically                           │
│     ├── Self-healing                                            │
│     └── Alerts on differences                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. ArgoCD Deep Dive

### 3.1 ArgoCD Architecture

```
ArgoCD Architecture:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    ArgoCD Server                        │    │
│  │  ┌─────────────────────────────────────────────────────┐│    │
│  │  │                                                      ││    │
│  │  │  ┌──────────┐  ┌──────────┐  ┌──────────────────┐   ││    │
│  │  │  │   API    │  │   Repo   │  │   Application    │   ││    │
│  │  │  │  Server  │  │  Server  │  │   Controller     │   ││    │
│  │  │  └──────────┘  └──────────┘  └──────────────────┘   ││    │
│  │  │       │              │               │              ││    │
│  │  │       │              │               │              ││    │
│  │  │       ▼              ▼               ▼              ││    │
│  │  │  ┌─────────────────────────────────────────────┐    ││    │
│  │  │  │              Redis (Cache)                  │    ││    │
│  │  │  └─────────────────────────────────────────────┘    ││    │
│  │  │                                                      ││    │
│  │  └─────────────────────────────────────────────────────┘│    │
│  └─────────────────────────────────────────────────────────┘    │
│           │                    │                                │
│           │                    │                                │
│           ▼                    ▼                                │
│  ┌──────────────────┐  ┌──────────────────┐                     │
│  │   Git Repos      │  │ Target Clusters  │                     │
│  │  (Source)        │  │ (Destination)    │                     │
│  └──────────────────┘  └──────────────────┘                     │
│                                                                 │
│  Components:                                                    │
│  ├── API Server: Exposes gRPC/REST API, Web UI                  │
│  ├── Repo Server: Clones repos, generates manifests             │
│  ├── Application Controller: Monitors apps, syncs state         │
│  └── Redis: Caching layer                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 Installing ArgoCD

```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for pods
kubectl wait --for=condition=Ready pods --all -n argocd --timeout=300s

# Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Access UI (port-forward)
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Or expose via Ingress
# Access at https://localhost:8080, login as 'admin'

# Install CLI
# Windows: choco install argocd-cli
# Mac: brew install argocd
# Linux: curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64

# Login with CLI
argocd login localhost:8080 --insecure

# Change password
argocd account update-password
```

### 3.3 ArgoCD Application

```yaml
# Simple Application
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  
  source:
    repoURL: https://github.com/myorg/myapp-manifests.git
    targetRevision: main
    path: environments/production
  
  destination:
    server: https://kubernetes.default.svc
    namespace: myapp
  
  syncPolicy:
    automated:
      prune: true        # Delete resources not in Git
      selfHeal: true     # Fix drift automatically
    syncOptions:
    - CreateNamespace=true
    - PruneLast=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m

---
# Application with Helm
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-ingress
  namespace: argocd
spec:
  project: default
  
  source:
    repoURL: https://kubernetes.github.io/ingress-nginx
    chart: ingress-nginx
    targetRevision: 4.7.1
    helm:
      releaseName: nginx-ingress
      values: |
        controller:
          replicaCount: 2
          service:
            type: LoadBalancer
          metrics:
            enabled: true
  
  destination:
    server: https://kubernetes.default.svc
    namespace: ingress-nginx
  
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true

---
# Application with Kustomize
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-production
  namespace: argocd
spec:
  project: default
  
  source:
    repoURL: https://github.com/myorg/myapp.git
    targetRevision: main
    path: deploy/overlays/production
    kustomize:
      images:
      - myapp=myregistry/myapp:v1.2.3
  
  destination:
    server: https://kubernetes.default.svc
    namespace: production
```

### 3.4 ArgoCD Projects

```yaml
# ArgoCD Project (RBAC)
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: team-payments
  namespace: argocd
spec:
  description: "Payments team applications"
  
  # Allowed source repositories
  sourceRepos:
  - https://github.com/myorg/payments-*
  - https://github.com/myorg/shared-libs
  
  # Allowed destination clusters and namespaces
  destinations:
  - namespace: payments-*
    server: https://kubernetes.default.svc
  - namespace: payments-*
    server: https://production-cluster.example.com
  
  # Allowed cluster resources
  clusterResourceWhitelist:
  - group: ''
    kind: Namespace
  - group: 'rbac.authorization.k8s.io'
    kind: ClusterRole
  - group: 'rbac.authorization.k8s.io'
    kind: ClusterRoleBinding
  
  # Denied namespace resources
  namespaceResourceBlacklist:
  - group: ''
    kind: ResourceQuota
  - group: ''
    kind: LimitRange
  
  # Roles within project
  roles:
  - name: developer
    description: Developer access
    policies:
    - p, proj:team-payments:developer, applications, get, team-payments/*, allow
    - p, proj:team-payments:developer, applications, sync, team-payments/*, allow
    groups:
    - payments-developers
  
  - name: admin
    description: Admin access
    policies:
    - p, proj:team-payments:admin, applications, *, team-payments/*, allow
    groups:
    - payments-admins
```

### 3.5 ArgoCD CLI Commands

```bash
# Applications
argocd app list
argocd app get myapp
argocd app create myapp \
  --repo https://github.com/myorg/repo.git \
  --path deploy \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace myapp

argocd app sync myapp
argocd app sync myapp --prune
argocd app wait myapp
argocd app delete myapp

# Diff and history
argocd app diff myapp
argocd app history myapp
argocd app rollback myapp 2

# Refresh
argocd app get myapp --refresh
argocd app get myapp --hard-refresh

# Projects
argocd proj list
argocd proj get team-payments
argocd proj create team-backend \
  --src https://github.com/myorg/* \
  --dest https://kubernetes.default.svc,backend

# Repos
argocd repo list
argocd repo add https://github.com/myorg/repo.git \
  --username git \
  --password $GITHUB_TOKEN

# Clusters
argocd cluster list
argocd cluster add production-context
argocd cluster rm https://production.example.com
```

---

## 4. Flux CD Deep Dive

### 4.1 Flux Architecture

```
Flux Architecture:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Flux Controllers:                                              │
│                                                                 │
│  ┌──────────────────┐    ┌──────────────────┐                   │
│  │  Source          │    │  Kustomize       │                   │
│  │  Controller      │───►│  Controller      │                   │
│  │  (Git, Helm,     │    │  (Apply          │                   │
│  │   OCI, Bucket)   │    │   manifests)     │                   │
│  └──────────────────┘    └──────────────────┘                   │
│           │                       │                             │
│           │                       ▼                             │
│           │              ┌──────────────────┐                   │
│           │              │  Helm            │                   │
│           │              │  Controller      │                   │
│           └─────────────►│  (Helm releases) │                   │
│                          └──────────────────┘                   │
│                                   │                             │
│                                   ▼                             │
│                          ┌──────────────────┐                   │
│                          │  Notification    │                   │
│                          │  Controller      │                   │
│                          │  (Alerts, Events)│                   │
│                          └──────────────────┘                   │
│                                   │                             │
│                          ┌──────────────────┐                   │
│                          │  Image           │                   │
│                          │  Automation      │                   │
│                          │  Controller      │                   │
│                          └──────────────────┘                   │
│                                                                 │
│  Key Resources:                                                 │
│  ├── GitRepository: Source from Git                             │
│  ├── HelmRepository: Source from Helm repo                      │
│  ├── Kustomization: Apply Kustomize overlays                    │
│  ├── HelmRelease: Deploy Helm charts                            │
│  ├── ImageRepository: Watch container registry                  │
│  ├── ImagePolicy: Define update rules                           │
│  └── ImageUpdateAutomation: Auto-update manifests               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 Installing Flux

```bash
# Install Flux CLI
# Windows: choco install flux
# Mac: brew install fluxcd/tap/flux
# Linux: curl -s https://fluxcd.io/install.sh | sudo bash

# Check prerequisites
flux check --pre

# Bootstrap Flux with GitHub
flux bootstrap github \
  --owner=myorg \
  --repository=fleet-infra \
  --branch=main \
  --path=clusters/production \
  --personal

# Bootstrap with Azure DevOps
flux bootstrap git \
  --url=https://dev.azure.com/myorg/myproject/_git/fleet-infra \
  --branch=main \
  --path=clusters/production \
  --token-auth

# Check installation
flux check

# View Flux resources
kubectl get all -n flux-system
```

### 4.3 Flux Resources

```yaml
# GitRepository Source
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: myapp
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/myorg/myapp
  ref:
    branch: main
  secretRef:
    name: github-credentials

---
# Kustomization (apply manifests)
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: myapp
  namespace: flux-system
spec:
  interval: 10m
  targetNamespace: myapp
  sourceRef:
    kind: GitRepository
    name: myapp
  path: ./deploy/overlays/production
  prune: true
  healthChecks:
  - apiVersion: apps/v1
    kind: Deployment
    name: myapp
    namespace: myapp
  timeout: 3m
  postBuild:
    substitute:
      ENVIRONMENT: production
      REPLICAS: "3"

---
# HelmRepository
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: HelmRepository
metadata:
  name: bitnami
  namespace: flux-system
spec:
  interval: 30m
  url: https://charts.bitnami.com/bitnami

---
# HelmRelease
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: redis
  namespace: redis
spec:
  interval: 5m
  chart:
    spec:
      chart: redis
      version: "17.x"
      sourceRef:
        kind: HelmRepository
        name: bitnami
        namespace: flux-system
  values:
    architecture: standalone
    auth:
      enabled: true
      existingSecret: redis-credentials
    master:
      persistence:
        size: 10Gi
  valuesFrom:
  - kind: ConfigMap
    name: redis-values
    optional: true

---
# Image automation
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImageRepository
metadata:
  name: myapp
  namespace: flux-system
spec:
  image: myregistry.azurecr.io/myapp
  interval: 1m
  secretRef:
    name: acr-credentials

---
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImagePolicy
metadata:
  name: myapp
  namespace: flux-system
spec:
  imageRepositoryRef:
    name: myapp
  policy:
    semver:
      range: ">=1.0.0"

---
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImageUpdateAutomation
metadata:
  name: myapp
  namespace: flux-system
spec:
  interval: 1m
  sourceRef:
    kind: GitRepository
    name: myapp
  git:
    checkout:
      ref:
        branch: main
    commit:
      author:
        name: flux
        email: flux@myorg.com
      messageTemplate: |
        Automated image update
        
        Automation: {{ .AutomationObject }}
        Images:
        {{ range .Updated.Images -}}
        - {{ .Repository }}:{{ .Tag }}
        {{ end -}}
    push:
      branch: main
  update:
    path: ./deploy
    strategy: Setters
```

### 4.4 Flux CLI Commands

```bash
# Check status
flux get all
flux get sources git
flux get kustomizations
flux get helmreleases

# Reconcile (force sync)
flux reconcile source git myapp
flux reconcile kustomization myapp
flux reconcile helmrelease redis

# Suspend/Resume
flux suspend kustomization myapp
flux resume kustomization myapp

# Logs
flux logs
flux logs --kind=Kustomization --name=myapp

# Export resources
flux export source git myapp > git-source.yaml
flux export kustomization myapp > kustomization.yaml

# Create resources
flux create source git myapp \
  --url=https://github.com/myorg/myapp \
  --branch=main \
  --interval=1m

flux create kustomization myapp \
  --source=myapp \
  --path=./deploy \
  --prune=true \
  --interval=10m

# Uninstall
flux uninstall
```

---

## 5. Repository Strategies

### 5.1 Mono-repo vs Multi-repo

```
Repository Strategies:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  MONO-REPO (Single Repository):                                 │
│                                                                 │
│  fleet-repo/                                                    │
│  ├── apps/                                                      │
│  │   ├── app1/                                                  │
│  │   │   ├── base/                                              │
│  │   │   └── overlays/                                          │
│  │   │       ├── dev/                                           │
│  │   │       ├── staging/                                       │
│  │   │       └── prod/                                          │
│  │   └── app2/                                                  │
│  ├── infrastructure/                                            │
│  │   ├── cert-manager/                                          │
│  │   ├── ingress-nginx/                                         │
│  │   └── monitoring/                                            │
│  └── clusters/                                                  │
│      ├── dev/                                                   │
│      ├── staging/                                               │
│      └── prod/                                                  │
│                                                                 │
│  Pros: Single place for all config, atomic changes              │
│  Cons: Large repo, access control challenges                    │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  MULTI-REPO (Separate Repositories):                            │
│                                                                 │
│  app1-config/     - App 1 manifests                             │
│  app2-config/     - App 2 manifests                             │
│  infra-config/    - Infrastructure components                   │
│  cluster-config/  - Cluster-level config                        │
│                                                                 │
│  Pros: Team autonomy, clear ownership, fine-grained access      │
│  Cons: Coordination overhead, harder to see full picture        │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  HYBRID (Recommended):                                          │
│                                                                 │
│  platform-config/        - Shared infrastructure                │
│  ├── infrastructure/                                            │
│  ├── clusters/                                                  │
│  └── apps/               - App of apps                          │
│                                                                 │
│  team-a-apps/            - Team A applications                  │
│  team-b-apps/            - Team B applications                  │
│                                                                 │
│  Pros: Balance of centralization and autonomy                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 Environment Promotion

```
Environment Promotion Strategies:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. BRANCH PER ENVIRONMENT                                      │
│                                                                 │
│     main ────────► dev cluster                                  │
│       │                                                         │
│       └─ staging ──► staging cluster                            │
│              │                                                  │
│              └─ production ──► prod cluster                     │
│                                                                 │
│     Promotion: Cherry-pick or merge between branches            │
│                                                                 │
│  2. PATH PER ENVIRONMENT (Recommended)                          │
│                                                                 │
│     main branch:                                                │
│     ├── base/           - Common configuration                  │
│     ├── overlays/                                               │
│     │   ├── dev/        - Dev overrides (image: v1.3.0)         │
│     │   ├── staging/    - Staging overrides (image: v1.2.0)     │
│     │   └── prod/       - Prod overrides (image: v1.1.0)        │
│                                                                 │
│     Promotion: Update image tag in overlay                      │
│                                                                 │
│  3. TAG/RELEASE BASED                                           │
│                                                                 │
│     dev: HEAD of main                                           │
│     staging: tag v1.2.0                                         │
│     prod: tag v1.1.0                                            │
│                                                                 │
│     Promotion: Create new tag, update ArgoCD targetRevision     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

Example Kustomize Structure:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  myapp/                                                         │
│  ├── base/                                                      │
│  │   ├── kustomization.yaml                                     │
│  │   ├── deployment.yaml                                        │
│  │   ├── service.yaml                                           │
│  │   └── configmap.yaml                                         │
│  └── overlays/                                                  │
│      ├── dev/                                                   │
│      │   ├── kustomization.yaml                                 │
│      │   └── patch-replicas.yaml  (replicas: 1)                 │
│      ├── staging/                                               │
│      │   ├── kustomization.yaml                                 │
│      │   └── patch-replicas.yaml  (replicas: 2)                 │
│      └── prod/                                                  │
│          ├── kustomization.yaml                                 │
│          ├── patch-replicas.yaml  (replicas: 5)                 │
│          └── patch-resources.yaml (more CPU/memory)             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. Secret Management

### 6.1 Sealed Secrets

```bash
# Install Sealed Secrets controller
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.24.0/controller.yaml

# Install kubeseal CLI
# Mac: brew install kubeseal
# Linux: wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.24.0/kubeseal-linux-amd64

# Create sealed secret
kubectl create secret generic my-secret \
  --from-literal=password=supersecret \
  --dry-run=client -o yaml | \
  kubeseal --format yaml > sealed-secret.yaml

# The sealed-secret.yaml can be committed to Git
# Controller decrypts it in cluster
```

```yaml
# Sealed Secret example
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  name: my-secret
  namespace: myapp
spec:
  encryptedData:
    password: AgBy3i4OJSWK+PiTySYZZA9rO43cGDEq...
  template:
    metadata:
      name: my-secret
      namespace: myapp
    type: Opaque
```

### 6.2 SOPS with Age/GPG

```bash
# Install SOPS
# Mac: brew install sops
# Linux: Download from https://github.com/getsops/sops/releases

# Generate age key
age-keygen -o key.txt
# Save public key for encryption
# Store private key securely

# Create .sops.yaml config
cat > .sops.yaml << EOF
creation_rules:
  - path_regex: .*\.enc\.yaml$
    age: age1ql3z7hjy54pw3hyww5ayyfg7zqgvc7w3j2elw8zmrj2kg5sfn9aqmcac8p
EOF

# Encrypt secret
sops --encrypt --in-place secrets.enc.yaml

# Decrypt (for local editing)
sops --decrypt secrets.enc.yaml

# With Flux - configure decryption
kubectl create secret generic sops-age \
  --namespace=flux-system \
  --from-file=age.agekey=key.txt
```

```yaml
# Flux Kustomization with SOPS
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: myapp
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: myapp
  path: ./deploy
  prune: true
  decryption:
    provider: sops
    secretRef:
      name: sops-age
```

### 6.3 External Secrets Operator

```yaml
# Install External Secrets Operator
# helm install external-secrets external-secrets/external-secrets -n external-secrets --create-namespace

# Azure Key Vault SecretStore
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: azure-keyvault
  namespace: myapp
spec:
  provider:
    azurekv:
      vaultUrl: "https://mykeyvault.vault.azure.net"
      authType: WorkloadIdentity
      serviceAccountRef:
        name: external-secrets-sa

---
# ExternalSecret
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: database-credentials
  namespace: myapp
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: azure-keyvault
    kind: SecretStore
  target:
    name: database-credentials
    creationPolicy: Owner
  data:
  - secretKey: username
    remoteRef:
      key: database-username
  - secretKey: password
    remoteRef:
      key: database-password

---
# ClusterSecretStore (cluster-wide)
apiVersion: external-secrets.io/v1beta1
kind: ClusterSecretStore
metadata:
  name: azure-keyvault-global
spec:
  provider:
    azurekv:
      vaultUrl: "https://mykeyvault.vault.azure.net"
      authType: ManagedIdentity
      identityId: "xxx-xxx-xxx"
```

---

## 7. Progressive Delivery

### 7.1 Argo Rollouts

```bash
# Install Argo Rollouts
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml

# Install kubectl plugin
# Mac: brew install argoproj/tap/kubectl-argo-rollouts
```

```yaml
# Canary Rollout
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp
  namespace: myapp
spec:
  replicas: 10
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myregistry/myapp:v1.0.0
        ports:
        - containerPort: 8080
  strategy:
    canary:
      steps:
      - setWeight: 5
      - pause: { duration: 2m }
      - setWeight: 20
      - pause: { duration: 5m }
      - setWeight: 50
      - pause: { duration: 10m }
      - setWeight: 80
      - pause: { duration: 5m }
      canaryService: myapp-canary
      stableService: myapp-stable
      trafficRouting:
        nginx:
          stableIngress: myapp-ingress
      analysis:
        templates:
        - templateName: success-rate
        startingStep: 2
        args:
        - name: service-name
          value: myapp-canary

---
# Analysis Template
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: success-rate
spec:
  args:
  - name: service-name
  metrics:
  - name: success-rate
    interval: 1m
    successCondition: result[0] >= 0.95
    failureCondition: result[0] < 0.90
    failureLimit: 3
    provider:
      prometheus:
        address: http://prometheus.monitoring:9090
        query: |
          sum(rate(http_requests_total{service="{{args.service-name}}",status!~"5.."}[5m])) /
          sum(rate(http_requests_total{service="{{args.service-name}}"}[5m]))

---
# Blue-Green Rollout
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp-bluegreen
spec:
  replicas: 5
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myregistry/myapp:v1.0.0
  strategy:
    blueGreen:
      activeService: myapp-active
      previewService: myapp-preview
      autoPromotionEnabled: false
      scaleDownDelaySeconds: 30
      prePromotionAnalysis:
        templates:
        - templateName: success-rate
        args:
        - name: service-name
          value: myapp-preview
```

### 7.2 Flagger (Progressive Delivery for Flux)

```yaml
# Install Flagger
# helm upgrade -i flagger flagger/flagger \
#   --namespace=flagger-system \
#   --set prometheus.install=true \
#   --set meshProvider=nginx

# Canary resource
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: myapp
  namespace: myapp
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  
  service:
    port: 80
    targetPort: 8080
    gateways:
    - public-gateway.istio-system.svc.cluster.local
    hosts:
    - myapp.example.com
  
  analysis:
    interval: 1m
    threshold: 5
    maxWeight: 50
    stepWeight: 10
    
    metrics:
    - name: request-success-rate
      thresholdRange:
        min: 99
      interval: 1m
    - name: request-duration
      thresholdRange:
        max: 500
      interval: 1m
    
    webhooks:
    - name: load-test
      url: http://flagger-loadtester.flagger-system/
      timeout: 5s
      metadata:
        cmd: "hey -z 1m -q 10 -c 2 http://myapp-canary.myapp:80/"
```

---

## 8. Multi-Cluster GitOps

### 8.1 ArgoCD Multi-Cluster

```yaml
# Register external cluster
# argocd cluster add production-context --name production

# ApplicationSet for multi-cluster
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: myapp
  namespace: argocd
spec:
  generators:
  - clusters:
      selector:
        matchLabels:
          environment: production
  template:
    metadata:
      name: 'myapp-{{name}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/myorg/myapp.git
        targetRevision: main
        path: deploy/overlays/{{metadata.labels.environment}}
      destination:
        server: '{{server}}'
        namespace: myapp
      syncPolicy:
        automated:
          prune: true
          selfHeal: true

---
# Matrix generator for environments × clusters
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: myapp-all
  namespace: argocd
spec:
  generators:
  - matrix:
      generators:
      - clusters:
          selector:
            matchLabels:
              argocd.argoproj.io/secret-type: cluster
      - list:
          elements:
          - app: frontend
            path: apps/frontend
          - app: backend
            path: apps/backend
  template:
    metadata:
      name: '{{name}}-{{app}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/myorg/apps.git
        path: '{{path}}/overlays/{{metadata.labels.environment}}'
        targetRevision: main
      destination:
        server: '{{server}}'
        namespace: '{{app}}'
```

### 8.2 Flux Multi-Cluster

```yaml
# clusters/production/flux-system/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- gotk-components.yaml
- gotk-sync.yaml
- ../base/infrastructure.yaml
- ../base/apps.yaml

---
# clusters/base/infrastructure.yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: infrastructure
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: ./infrastructure
  prune: true
  wait: true

---
# clusters/base/apps.yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: apps
  namespace: flux-system
spec:
  interval: 10m
  dependsOn:
  - name: infrastructure
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: ./apps
  prune: true
  postBuild:
    substituteFrom:
    - kind: ConfigMap
      name: cluster-vars
```

---

## 9. GitOps Best Practices

```
GitOps Best Practices:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. STRUCTURE                                                   │
│     ├── Separate app code from config repos                     │
│     ├── Use Kustomize or Helm for templating                    │
│     ├── Clear directory structure (base/overlays)               │
│     └── Document repo structure in README                       │
│                                                                 │
│  2. SECURITY                                                    │
│     ├── Never commit plain secrets to Git                       │
│     ├── Use Sealed Secrets, SOPS, or External Secrets           │
│     ├── Limit ArgoCD/Flux RBAC per team                         │
│     └── Enable audit logging                                    │
│                                                                 │
│  3. WORKFLOW                                                    │
│     ├── Pull requests for all changes                           │
│     ├── Require reviews and approvals                           │
│     ├── Run validation in CI (kubeval, conftest)                │
│     └── Use branch protection                                   │
│                                                                 │
│  4. DEPLOYMENTS                                                 │
│     ├── Enable auto-sync with prune and selfHeal                │
│     ├── Use progressive delivery for production                 │
│     ├── Define health checks for all apps                       │
│     └── Set sync timeouts and retry policies                    │
│                                                                 │
│  5. OBSERVABILITY                                               │
│     ├── Monitor sync status and health                          │
│     ├── Alert on sync failures                                  │
│     ├── Track deployment frequency                              │
│     └── Measure lead time for changes                           │
│                                                                 │
│  6. DISASTER RECOVERY                                           │
│     ├── Backup GitOps tool configuration                        │
│     ├── Document recovery procedure                             │
│     ├── Test recreation from Git regularly                      │
│     └── Keep secrets backup separate                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Summary

GitOps provides a modern approach to continuous delivery:

1. **Git as Source of Truth**: All infrastructure and app configs in Git
2. **Pull-Based Deployment**: Operators pull from Git, don't push to cluster
3. **ArgoCD**: Full-featured GitOps with UI, SSO, RBAC, multi-cluster
4. **Flux**: Lightweight, toolkit approach, native Kubernetes
5. **Secret Management**: Sealed Secrets, SOPS, External Secrets
6. **Progressive Delivery**: Canary, Blue-Green with Argo Rollouts/Flagger
7. **Multi-Cluster**: ApplicationSets (ArgoCD), Kustomizations (Flux)

Choose ArgoCD for a comprehensive solution with UI, or Flux for a modular, Kubernetes-native approach.

---

**Next Part**: [Part 12: Service Mesh with Istio](./DevOps-Complete-Reference-Guide-Part12-ServiceMesh.md)
