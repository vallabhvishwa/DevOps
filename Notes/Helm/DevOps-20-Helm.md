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
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                                                                 â”‚
â”‚  CHART: Package of Kubernetes resources                         â”‚
â”‚  â”œâ”€â”€ Templates + Values = Rendered manifests                   â”‚
â”‚  â””â”€â”€ Versioned, shareable, reusable                             â”‚
â”‚                                                                 â”‚
â”‚  RELEASE: Instance of a chart in a cluster                      â”‚
â”‚  â”œâ”€â”€ helm install myrelease ./mychart                           â”‚
â”‚  â””â”€â”€ Multiple releases of same chart possible                   â”‚
â”‚                                                                 â”‚
â”‚  REPOSITORY: Collection of charts                               â”‚
â”‚  â”œâ”€â”€ helm repo add bitnami https://charts.bitnami.com/bitnami   â”‚
â”‚  â””â”€â”€ Public or private                                          â”‚
â”‚                                                                 â”‚
â”‚  Workflow:                                                      â”‚
â”‚  Chart + Values â”€â”€â–º helm template â”€â”€â–º K8s Manifests             â”‚
â”‚                              â”‚                                  â”‚
â”‚                              â–¼                                  â”‚
â”‚                     kubectl apply (via helm)                    â”‚
â”‚                                                                 â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

---

## 2. Chart Structure

```
mychart/
â”œâ”€â”€ Chart.yaml          # Chart metadata
â”œâ”€â”€ values.yaml         # Default values
â”œâ”€â”€ values-prod.yaml    # Environment overrides
â”œâ”€â”€ charts/             # Dependencies
â”œâ”€â”€ templates/          # Kubernetes templates
â”‚   â”œâ”€â”€ NOTES.txt       # Post-install notes
â”‚   â”œâ”€â”€ _helpers.tpl    # Template helpers
â”‚   â”œâ”€â”€ deployment.yaml
â”‚   â”œâ”€â”€ service.yaml
â”‚   â”œâ”€â”€ ingress.yaml
â”‚   â”œâ”€â”€ configmap.yaml
â”‚   â”œâ”€â”€ secret.yaml
â”‚   â”œâ”€â”€ hpa.yaml
â”‚   â””â”€â”€ tests/
â”‚       â””â”€â”€ test-connection.yaml
â””â”€â”€ .helmignore         # Files to ignore
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
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                                                                 â”‚
â”‚  1. NAMING                                                      â”‚
â”‚     â”œâ”€â”€ Use include for generated names                         â”‚
â”‚     â”œâ”€â”€ Keep names under 63 characters                          â”‚
â”‚     â””â”€â”€ Use consistent labels                                   â”‚
â”‚                                                                 â”‚
â”‚  2. VALUES                                                      â”‚
â”‚     â”œâ”€â”€ Document all values in values.yaml                      â”‚
â”‚     â”œâ”€â”€ Use sensible defaults                                   â”‚
â”‚     â”œâ”€â”€ Avoid deeply nested structures                          â”‚
â”‚     â””â”€â”€ Use values-{env}.yaml for environments                  â”‚
â”‚                                                                 â”‚
â”‚  3. TEMPLATES                                                   â”‚
â”‚     â”œâ”€â”€ Use _helpers.tpl for reusable functions                 â”‚
â”‚     â”œâ”€â”€ Quote strings with {{ .Value | quote }}                 â”‚
â”‚     â”œâ”€â”€ Use nindent for proper indentation                      â”‚
â”‚     â””â”€â”€ Handle empty values gracefully                          â”‚
â”‚                                                                 â”‚
â”‚  4. SECURITY                                                    â”‚
â”‚     â”œâ”€â”€ Never hardcode secrets in values                        â”‚
â”‚     â”œâ”€â”€ Use external secrets (Vault, External Secrets)          â”‚
â”‚     â””â”€â”€ Validate inputs                                         â”‚
â”‚                                                                 â”‚
â”‚  5. TESTING                                                     â”‚
â”‚     â”œâ”€â”€ helm lint before committing                             â”‚
â”‚     â”œâ”€â”€ helm template to verify output                          â”‚
â”‚     â”œâ”€â”€ Add helm test templates                                 â”‚
â”‚     â””â”€â”€ Test with different value combinations                  â”‚
â”‚                                                                 â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
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

**See also:** [Helm Enterprise Patterns](./Helm-Enterprise-Patterns.md) | [Helm Cheatsheet](../Cheatsheets/Helm-Cheatsheet.md) | [Databases](../Databases/DevOps-21-Databases.md)
