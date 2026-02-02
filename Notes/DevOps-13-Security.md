# DevOps Engineer's Complete Reference Guide
# Part 13: Advanced Security

---

## Table of Contents

1. [Security Fundamentals](#1-security-fundamentals)
2. [Zero Trust Architecture](#2-zero-trust-architecture)
3. [Container Security](#3-container-security)
4. [Supply Chain Security](#4-supply-chain-security)
5. [Kubernetes Security](#5-kubernetes-security)
6. [Secrets Management](#6-secrets-management)
7. [Network Security](#7-network-security)
8. [Compliance and Auditing](#8-compliance-and-auditing)
9. [Security Scanning Tools](#9-security-scanning-tools)

---

## 1. Security Fundamentals

### 1.1 Defense in Depth

```
Defense in Depth Strategy:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Layer 1: PERIMETER                                             │
│  ├── Firewalls, WAF                                             │
│  ├── DDoS protection                                            │
│  └── Network segmentation                                       │
│                                                                 │
│  Layer 2: NETWORK                                               │
│  ├── VPN, Private endpoints                                     │
│  ├── Network policies                                           │
│  └── mTLS                                                       │
│                                                                 │
│  Layer 3: IDENTITY                                              │
│  ├── Authentication (MFA)                                       │
│  ├── Authorization (RBAC)                                       │
│  └── Identity federation                                        │
│                                                                 │
│  Layer 4: APPLICATION                                           │
│  ├── Input validation                                           │
│  ├── Secure coding                                              │
│  └── API security                                               │
│                                                                 │
│  Layer 5: DATA                                                  │
│  ├── Encryption at rest                                         │
│  ├── Encryption in transit                                      │
│  └── Key management                                             │
│                                                                 │
│  Layer 6: DETECTION & RESPONSE                                  │
│  ├── Logging and monitoring                                     │
│  ├── Intrusion detection                                        │
│  └── Incident response                                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Security Principles

```
Core Security Principles:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  LEAST PRIVILEGE                                                │
│  └── Grant minimum permissions needed                           │
│                                                                 │
│  SEPARATION OF DUTIES                                           │
│  └── No single person controls entire process                   │
│                                                                 │
│  DEFENSE IN DEPTH                                               │
│  └── Multiple layers of security                                │
│                                                                 │
│  FAIL SECURE                                                    │
│  └── Deny access on error, not allow                            │
│                                                                 │
│  TRUST BUT VERIFY → NEVER TRUST, ALWAYS VERIFY                  │
│  └── Zero trust model                                           │
│                                                                 │
│  SECURITY BY DEFAULT                                            │
│  └── Secure out of the box                                      │
│                                                                 │
│  AUDITABILITY                                                   │
│  └── Log all security-relevant actions                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Zero Trust Architecture

```
Zero Trust Principles:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  "Never trust, always verify"                                   │
│                                                                 │
│  1. VERIFY EXPLICITLY                                           │
│     ├── Authenticate every request                              │
│     ├── Validate identity + device + context                    │
│     └── Continuous verification, not just at entry              │
│                                                                 │
│  2. USE LEAST PRIVILEGE ACCESS                                  │
│     ├── Just-in-time access                                     │
│     ├── Just-enough access                                      │
│     └── Risk-based adaptive policies                            │
│                                                                 │
│  3. ASSUME BREACH                                               │
│     ├── Minimize blast radius                                   │
│     ├── Segment access                                          │
│     ├── End-to-end encryption                                   │
│     └── Use analytics for visibility                            │
│                                                                 │
│  Implementation in Kubernetes:                                  │
│  ├── Network Policies (deny-all default)                        │
│  ├── mTLS between services (Istio)                              │
│  ├── Pod Security Standards                                     │
│  ├── RBAC with minimal permissions                              │
│  ├── Workload Identity for cloud access                         │
│  └── Runtime security monitoring                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Container Security

### 3.1 Secure Base Images

```dockerfile
# Bad: Using latest tag
FROM node:latest

# Good: Specific version, minimal image
FROM node:20-alpine3.18

# Better: Distroless (no shell, minimal attack surface)
FROM gcr.io/distroless/nodejs20-debian12

# Best practices in Dockerfile
FROM node:20-alpine3.18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Production image
FROM gcr.io/distroless/nodejs20-debian12
WORKDIR /app

# Non-root user
USER 1000

# Copy only necessary files
COPY --from=builder --chown=1000:1000 /app/dist ./dist
COPY --from=builder --chown=1000:1000 /app/node_modules ./node_modules

# Healthcheck
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js

# No secrets in image
# ENV DATABASE_PASSWORD=xxx  # NEVER DO THIS

EXPOSE 8080
CMD ["dist/server.js"]
```

### 3.2 Image Scanning

```bash
# Trivy (open source)
trivy image myapp:latest
trivy image --severity HIGH,CRITICAL myapp:latest
trivy image --exit-code 1 --severity CRITICAL myapp:latest

# Azure Container Registry scanning
az acr config content-trust update --registry myacr --status enabled

# Snyk
snyk container test myapp:latest
snyk container monitor myapp:latest

# Grype
grype myapp:latest

# In CI/CD pipeline
- name: Scan image
  run: |
    trivy image --exit-code 1 --severity CRITICAL $IMAGE
    if [ $? -ne 0 ]; then
      echo "Critical vulnerabilities found!"
      exit 1
    fi
```

### 3.3 Container Runtime Security

```yaml
# Pod Security Context
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
    fsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: myapp:latest
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
          - ALL
    volumeMounts:
    - name: tmp
      mountPath: /tmp
    - name: cache
      mountPath: /app/cache
  volumes:
  - name: tmp
    emptyDir: {}
  - name: cache
    emptyDir: {}
```

---

## 4. Supply Chain Security

### 4.1 SLSA Framework

```
SLSA (Supply-chain Levels for Software Artifacts):
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  LEVEL 1: Documentation                                         │
│  ├── Build process documented                                   │
│  └── Provenance available                                       │
│                                                                 │
│  LEVEL 2: Build Service                                         │
│  ├── Build by CI/CD service                                     │
│  ├── Signed provenance                                          │
│  └── Basic tampering resistance                                 │
│                                                                 │
│  LEVEL 3: Hardened Builds                                       │
│  ├── Isolated builds                                            │
│  ├── Non-falsifiable provenance                                 │
│  └── Prevents unauthorized changes                              │
│                                                                 │
│  LEVEL 4: Two-Party Review                                      │
│  ├── All changes reviewed                                       │
│  ├── Hermetic builds                                            │
│  └── Dependencies verified                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 SBOM (Software Bill of Materials)

```bash
# Generate SBOM with Syft
syft myapp:latest -o spdx-json > sbom.spdx.json
syft myapp:latest -o cyclonedx-json > sbom.cyclonedx.json

# Scan SBOM for vulnerabilities
grype sbom:sbom.spdx.json

# Trivy SBOM generation
trivy image --format spdx-json --output sbom.json myapp:latest
```

### 4.3 Image Signing

```bash
# Cosign (Sigstore)
# Generate key pair
cosign generate-key-pair

# Sign image
cosign sign --key cosign.key myregistry/myapp:v1.0.0

# Sign with keyless (GitHub OIDC)
cosign sign myregistry/myapp:v1.0.0

# Verify signature
cosign verify --key cosign.pub myregistry/myapp:v1.0.0

# Kubernetes admission with Kyverno
```

```yaml
# Kyverno policy - require signed images
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: verify-image-signature
spec:
  validationFailureAction: Enforce
  background: false
  rules:
  - name: verify-signature
    match:
      any:
      - resources:
          kinds:
          - Pod
    verifyImages:
    - imageReferences:
      - "myregistry/*"
      attestors:
      - entries:
        - keys:
            publicKeys: |-
              -----BEGIN PUBLIC KEY-----
              MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE...
              -----END PUBLIC KEY-----
```

---

## 5. Kubernetes Security

### 5.1 Pod Security Standards

```yaml
# Namespace with Pod Security Standards
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/enforce-version: latest
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted

# Levels:
# - privileged: No restrictions
# - baseline: Minimal restrictions, prevent known privilege escalations
# - restricted: Highly restricted, best practices

---
# Pod that passes restricted level
apiVersion: v1
kind: Pod
metadata:
  name: restricted-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 65534
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: myapp:latest
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop: ["ALL"]
      readOnlyRootFilesystem: true
```

### 5.2 RBAC Best Practices

```yaml
# Principle of least privilege
# Bad: Cluster-admin for developers
# Good: Namespace-scoped, minimal permissions

# Role for developers
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log", "pods/exec"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "update"]  # No create/delete
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list"]
# No access to secrets!

---
# RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-binding
  namespace: dev
subjects:
- kind: Group
  name: developers
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io

---
# Audit RBAC with kubectl
# kubectl auth can-i --list --as=system:serviceaccount:default:myapp
# kubectl auth can-i create pods --as=developer@example.com
```

### 5.3 Network Policies

```yaml
# Default deny all
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

---
# Allow specific traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
  - to:  # Allow DNS
    - namespaceSelector: {}
      podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
    - protocol: UDP
      port: 53
```

---

## 6. Secrets Management

### 6.1 Azure Key Vault with AKS

```yaml
# SecretProviderClass for Azure Key Vault
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: azure-keyvault-secrets
spec:
  provider: azure
  parameters:
    usePodIdentity: "false"
    useVMManagedIdentity: "true"
    userAssignedIdentityID: "<managed-identity-client-id>"
    keyvaultName: "mykeyvault"
    objects: |
      array:
        - |
          objectName: database-password
          objectType: secret
        - |
          objectName: api-key
          objectType: secret
    tenantId: "<tenant-id>"
  secretObjects:  # Sync to Kubernetes secret
  - secretName: app-secrets
    type: Opaque
    data:
    - objectName: database-password
      key: db-password

---
# Pod using secrets
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: myapp:latest
    volumeMounts:
    - name: secrets-store
      mountPath: "/mnt/secrets"
      readOnly: true
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secrets
          key: db-password
  volumes:
  - name: secrets-store
    csi:
      driver: secrets-store.csi.k8s.io
      readOnly: true
      volumeAttributes:
        secretProviderClass: azure-keyvault-secrets
```

### 6.2 Secret Rotation

```python
# Application with dynamic secret refresh
import os
import time
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler

class SecretReloader(FileSystemEventHandler):
    def __init__(self, app):
        self.app = app
    
    def on_modified(self, event):
        if event.src_path == "/mnt/secrets/database-password":
            new_password = open(event.src_path).read().strip()
            self.app.update_db_connection(new_password)
            print("Database password rotated")

# Watch for secret changes
observer = Observer()
observer.schedule(SecretReloader(app), "/mnt/secrets", recursive=False)
observer.start()
```

---

## 7. Network Security

### 7.1 Private Clusters

```bash
# Create private AKS cluster
az aks create \
  --resource-group myRG \
  --name myAKS \
  --enable-private-cluster \
  --private-dns-zone system \
  --network-plugin azure \
  --vnet-subnet-id $SUBNET_ID \
  --enable-managed-identity

# Access via private endpoint or VPN
```

### 7.2 Web Application Firewall

```yaml
# Azure Application Gateway WAF
# Configured via Terraform or ARM

# NGINX Ingress with ModSecurity
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-configuration
  namespace: ingress-nginx
data:
  enable-modsecurity: "true"
  enable-owasp-modsecurity-crs: "true"
  modsecurity-snippet: |
    SecRuleEngine On
    SecRequestBodyAccess On
    SecAuditEngine RelevantOnly
    SecAuditLogParts ABDEFHIJZ
```

---

## 8. Compliance and Auditing

### 8.1 Audit Logging

```yaml
# Kubernetes audit policy
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
# Log all requests to secrets at metadata level
- level: Metadata
  resources:
  - group: ""
    resources: ["secrets"]

# Log pod exec/attach at RequestResponse level
- level: RequestResponse
  resources:
  - group: ""
    resources: ["pods/exec", "pods/attach"]

# Log all changes
- level: RequestResponse
  verbs: ["create", "update", "patch", "delete"]

# Default: log at metadata level
- level: Metadata
```

### 8.2 Policy Enforcement

```yaml
# OPA Gatekeeper constraint
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: require-team-label
spec:
  match:
    kinds:
    - apiGroups: [""]
      kinds: ["Namespace"]
  parameters:
    labels: ["team", "environment"]

---
# Kyverno policy examples
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-requests-limits
spec:
  validationFailureAction: Enforce
  rules:
  - name: validate-resources
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: "CPU and memory requests/limits are required"
      pattern:
        spec:
          containers:
          - resources:
              requests:
                memory: "?*"
                cpu: "?*"
              limits:
                memory: "?*"
```

---

## 9. Security Scanning Tools

```
Security Tool Categories:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  STATIC ANALYSIS (SAST):                                        │
│  ├── SonarQube          - Code quality & security               │
│  ├── Semgrep            - Pattern-based scanning                │
│  ├── Checkov            - IaC security                          │
│  └── Snyk Code          - Real-time code analysis               │
│                                                                 │
│  CONTAINER SCANNING:                                            │
│  ├── Trivy              - Vulnerabilities, misconfig            │
│  ├── Grype              - Vulnerability scanner                 │
│  ├── Snyk Container     - Container vulnerabilities             │
│  └── Clair              - Container vulnerability DB            │
│                                                                 │
│  KUBERNETES SECURITY:                                           │
│  ├── kube-bench         - CIS benchmark checks                  │
│  ├── kube-hunter        - Penetration testing                   │
│  ├── Kubescape          - NSA/CISA hardening                    │
│  └── Polaris            - Best practices validation             │
│                                                                 │
│  RUNTIME SECURITY:                                              │
│  ├── Falco              - Runtime threat detection              │
│  ├── Sysdig             - Container security platform           │
│  └── Aqua Security      - Full lifecycle security               │
│                                                                 │
│  SECRET SCANNING:                                               │
│  ├── TruffleHog         - Git secret scanning                   │
│  ├── GitLeaks           - Secret detection                      │
│  └── detect-secrets     - Pre-commit secret detection           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```bash
# Quick security assessment
# CIS Kubernetes Benchmark
kube-bench run --targets node,master

# Kubescape scan
kubescape scan framework nsa --submit

# Falco runtime monitoring
helm install falco falcosecurity/falco --namespace falco
```

---

## Summary

Security in DevOps requires a comprehensive approach:

1. **Defense in Depth**: Multiple layers of security
2. **Zero Trust**: Never trust, always verify
3. **Container Security**: Minimal images, scanning, runtime protection
4. **Supply Chain**: SBOM, signing, provenance verification
5. **Kubernetes Security**: Pod Security Standards, RBAC, Network Policies
6. **Secrets Management**: External vaults, rotation, no hardcoding
7. **Compliance**: Audit logging, policy enforcement
8. **Continuous Scanning**: SAST, container scanning, runtime detection

Security is not a one-time activity but a continuous process integrated into every stage of the DevOps lifecycle.

---

**Next Part**: [Part 14: Chaos Engineering](./DevOps-Complete-Reference-Guide-Part14-ChaosEngineering.md)
