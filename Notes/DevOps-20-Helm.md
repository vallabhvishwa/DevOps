# DevOps Engineer's Complete Reference Guide
# Part 20: Helm Deep Dive

---

## Table of Contents

1. [Helm Fundamentals](#1-helm-fundamentals)
2. [Chart Structure](#2-chart-structure)
3. [Templating](#3-templating)
4. [Chart Development](#4-chart-development)
5. [Helm Commands](#5-helm-commands)
6. [Best Practices](#6-best-practices)

---

## 1. Helm Fundamentals

```
Helm Concepts:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  CHART: Package of Kubernetes resources                         │
│  ├── Templates + Values = Rendered manifests                   │
│  └── Versioned, shareable, reusable                             │
│                                                                 │
│  RELEASE: Instance of a chart in a cluster                      │
│  ├── helm install myrelease ./mychart                           │
│  └── Multiple releases of same chart possible                   │
│                                                                 │
│  REPOSITORY: Collection of charts                               │
│  ├── helm repo add bitnami https://charts.bitnami.com/bitnami   │
│  └── Public or private                                          │
│                                                                 │
│  Workflow:                                                      │
│  Chart + Values ──► helm template ──► K8s Manifests             │
│                              │                                  │
│                              ▼                                  │
│                     kubectl apply (via helm)                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Chart Structure

```
mychart/
├── Chart.yaml          # Chart metadata
├── values.yaml         # Default values
├── values-prod.yaml    # Environment overrides
├── charts/             # Dependencies
├── templates/          # Kubernetes templates
│   ├── NOTES.txt       # Post-install notes
│   ├── _helpers.tpl    # Template helpers
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── hpa.yaml
│   └── tests/
│       └── test-connection.yaml
└── .helmignore         # Files to ignore
```

```yaml
# Chart.yaml
apiVersion: v2
name: myapp
description: My application Helm chart
type: application
version: 1.0.0        # Chart version
appVersion: "2.0.0"   # App version
dependencies:
  - name: postgresql
    version: "12.x.x"
    repository: https://charts.bitnami.com/bitnami
    condition: postgresql.enabled
```

---

## 3. Templating

### 3.1 Basic Syntax

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "myapp.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "myapp.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          ports:
            - containerPort: {{ .Values.containerPort }}
          {{- if .Values.resources }}
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          {{- end }}
          env:
            {{- range $key, $value := .Values.env }}
            - name: {{ $key }}
              value: {{ $value | quote }}
            {{- end }}
```

### 3.2 Helper Templates

```yaml
# templates/_helpers.tpl
{{- define "myapp.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{- define "myapp.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}

{{- define "myapp.labels" -}}
helm.sh/chart: {{ .Chart.Name }}-{{ .Chart.Version }}
app.kubernetes.io/name: {{ include "myapp.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{- define "myapp.selectorLabels" -}}
app.kubernetes.io/name: {{ include "myapp.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}
```

### 3.3 Conditionals and Loops

```yaml
# Conditionals
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
# ...
{{- end }}

# With default
image: {{ .Values.image.tag | default "latest" }}

# Ternary
replicas: {{ ternary 3 1 .Values.production }}

# Range (loop)
env:
{{- range $key, $val := .Values.env }}
  - name: {{ $key }}
    value: {{ $val | quote }}
{{- end }}

# Range with index
{{- range $index, $host := .Values.ingress.hosts }}
  - host: {{ $host }}
{{- end }}
```

---

## 4. Chart Development

```yaml
# values.yaml
replicaCount: 2

image:
  repository: myapp
  tag: ""
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  className: nginx
  hosts:
    - host: myapp.local
      paths:
        - path: /
          pathType: Prefix

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 256Mi

autoscaling:
  enabled: false
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80

postgresql:
  enabled: true
  auth:
    database: myapp
```

---

## 5. Helm Commands

```bash
# Repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo nginx

# Install/Upgrade
helm install myrelease ./mychart
helm install myrelease ./mychart -f values-prod.yaml
helm install myrelease ./mychart --set replicaCount=3
helm upgrade myrelease ./mychart
helm upgrade --install myrelease ./mychart  # Install or upgrade

# Debugging
helm template myrelease ./mychart           # Render locally
helm template myrelease ./mychart --debug   # With debug
helm lint ./mychart                         # Validate chart
helm get manifest myrelease                 # See deployed manifests
helm get values myrelease                   # See used values

# Management
helm list                    # List releases
helm history myrelease       # Release history
helm rollback myrelease 1    # Rollback to revision
helm uninstall myrelease     # Delete release

# Dependencies
helm dependency update ./mychart
helm dependency build ./mychart

# Package & Push
helm package ./mychart
helm push mychart-1.0.0.tgz oci://myregistry.azurecr.io/helm
```

---

## 6. Best Practices

```
Helm Best Practices:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. NAMING                                                      │
│     ├── Use include for generated names                         │
│     ├── Keep names under 63 characters                          │
│     └── Use consistent labels                                   │
│                                                                 │
│  2. VALUES                                                      │
│     ├── Document all values in values.yaml                      │
│     ├── Use sensible defaults                                   │
│     ├── Avoid deeply nested structures                          │
│     └── Use values-{env}.yaml for environments                  │
│                                                                 │
│  3. TEMPLATES                                                   │
│     ├── Use _helpers.tpl for reusable functions                 │
│     ├── Quote strings with {{ .Value | quote }}                 │
│     ├── Use nindent for proper indentation                      │
│     └── Handle empty values gracefully                          │
│                                                                 │
│  4. SECURITY                                                    │
│     ├── Never hardcode secrets in values                        │
│     ├── Use external secrets (Vault, External Secrets)          │
│     └── Validate inputs                                         │
│                                                                 │
│  5. TESTING                                                     │
│     ├── helm lint before committing                             │
│     ├── helm template to verify output                          │
│     ├── Add helm test templates                                 │
│     └── Test with different value combinations                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Summary

Helm is essential for Kubernetes package management:

1. **Charts**: Reusable packages of K8s resources
2. **Templating**: Go templates with values substitution
3. **Releases**: Versioned deployments with rollback
4. **Dependencies**: Compose complex applications

Master Helm for efficient Kubernetes application management.

---

**Next Part**: [Part 21: Database Operations](./DevOps-Complete-Reference-Guide-Part21-Databases.md)
