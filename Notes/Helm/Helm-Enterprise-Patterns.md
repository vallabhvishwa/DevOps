# Helm Enterprise Patterns

> **last_reviewed:** 2026-06-30  
> **Prerequisites:** [Helm Deep Dive](DevOps-20-Helm.md) | **Project:** [PetClinic](../../Projects/PetClinic.md) | **Troubleshooting:** [Kubernetes](../../Troubleshooting/Kubernetes/02-Kubernetes-Troubleshooting.md)

Enterprise Helm charts (multi-service apps, config templates, multi-environment) follow patterns common in production AKS deployments.

---

## 1. Multi-environment values

```
helm/petclinic/
├── Chart.yaml
├── values.yaml              # Safe defaults only
├── values-dev.yaml
├── values-staging.yaml
├── values-prod.yaml
└── templates/
```

```bash
# Dev
helm upgrade --install petclinic ./helm/petclinic \
  -f values.yaml -f values-dev.yaml -n petclinic-dev

# Prod — never use --set for secrets
helm upgrade --install petclinic ./helm/petclinic \
  -f values.yaml -f values-prod.yaml -n petclinic-prod
```

```yaml
# values.yaml — minimal defaults
global:
  imageRegistry: myregistry.azurecr.io
  pullPolicy: IfNotPresent

apiGateway:
  enabled: true
  replicaCount: 1
  image:
    repository: petclinic/api-gateway
    tag: ""

# values-prod.yaml — overrides only
apiGateway:
  replicaCount: 3
  resources:
    requests:
      cpu: 250m
      memory: 512Mi
    limits:
      cpu: "1"
      memory: 1Gi

ingress:
  enabled: true
  host: petclinic.example.com
  tls:
    enabled: true
```

---

## 2. `_helpers.tpl` — naming and labels

```yaml
{{- define "petclinic.fullname" -}}
{{- printf "%s-%s" .Release.Name .Chart.Name | trunc 63 | trimSuffix "-" }}
{{- end }}

{{- define "petclinic.labels" -}}
helm.sh/chart: {{ .Chart.Name }}-{{ .Chart.Version | replace "+" "_" }}
app.kubernetes.io/name: {{ .Chart.Name }}
app.kubernetes.io/instance: {{ .Release.Name }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
environment: {{ .Values.global.environment | default "dev" }}
{{- end }}

{{- define "petclinic.image" -}}
{{- printf "%s/%s:%s" .Values.global.imageRegistry .repository (.tag | default .Chart.AppVersion) }}
{{- end }}
```

Usage in deployment:

```yaml
metadata:
  name: {{ include "petclinic.fullname" . }}-gateway
  labels:
    {{- include "petclinic.labels" . | nindent 4 }}
spec:
  template:
    spec:
      containers:
        - name: api-gateway
          image: {{ include "petclinic.image" (dict "Values" .Values "Chart" .Chart "repository" .Values.apiGateway.image.repository "tag" .Values.apiGateway.image.tag) }}
```

---

## 3. ConfigMap from template files (enterprise pattern)

Pattern used when app config is large and environment-specific:

```
helm/petclinic/
├── configurations/
│   ├── gateway.config.template
│   └── customers.config.template
└── templates/
    └── configmap.yaml
```

```yaml
# templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "petclinic.fullname" . }}-gateway-config
data:
  application.yml: |
{{ tpl (.Files.Get "configurations/gateway.config.template") . | indent 4 }}
```

Template file uses Helm variables:

```yaml
# configurations/gateway.config.template
spring:
  cloud:
    config:
      uri: http://{{ include "petclinic.fullname" . }}-config:8888
  datasource:
    url: jdbc:postgresql://{{ .Values.customers.db.host }}:5432/{{ .Values.customers.db.name }}
```

**Debug rendered config:**

```bash
helm template petclinic ./helm/petclinic -f values-dev.yaml \
  --show-only templates/configmap.yaml
```

---

## 4. Conditional services with `enabled` flags

```yaml
{{- if .Values.customers.enabled }}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "petclinic.fullname" . }}-customers
# ...
{{- end }}
```

```yaml
# values.yaml
customers:
  enabled: true
genai:
  enabled: false   # disable in dev to save cost
```

---

## 5. Secrets — never in values.yaml

```yaml
# BAD — never commit
database:
  password: "SuperSecret123"

# GOOD — External Secrets Operator or CSI driver
database:
  existingSecret: petclinic-db-credentials
  secretKey: password
```

```yaml
# templates/deployment.yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: {{ .Values.database.existingSecret }}
        key: {{ .Values.database.secretKey }}
```

See [Azure Private Link & Key Vault](../Azure/Azure-Private-Link-KeyVault-CICD.md).

---

## 6. Hooks — DB migration job

```yaml
# templates/db-migrate-job.yaml
{{- if .Values.migration.enabled }}
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "petclinic.fullname" . }}-migrate
  annotations:
    "helm.sh/hook": pre-install,pre-upgrade
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": before-hook-creation
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: flyway
          image: flyway/flyway:10
          args: ["migrate"]
          envFrom:
            - secretRef:
                name: {{ .Values.migration.secretName }}
{{- end }}
```

---

## 7. HPA and PDB

```yaml
{{- if .Values.apiGateway.autoscaling.enabled }}
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: {{ include "petclinic.fullname" . }}-gateway
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ include "petclinic.fullname" . }}-gateway
  minReplicas: {{ .Values.apiGateway.autoscaling.minReplicas }}
  maxReplicas: {{ .Values.apiGateway.autoscaling.maxReplicas }}
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: {{ .Values.apiGateway.autoscaling.targetCPU }}
{{- end }}
```

---

## 8. Chart testing workflow

```bash
# Lint
helm lint ./helm/petclinic -f values-dev.yaml

# Dry-run against cluster
helm upgrade --install petclinic ./helm/petclinic \
  -f values-dev.yaml --dry-run --debug

# Diff (plugin: helm-diff)
helm diff upgrade petclinic ./helm/petclinic -f values-prod.yaml

# OCI push to ACR
helm package ./helm/petclinic
helm push petclinic-1.0.0.tgz oci://myregistry.azurecr.io/helm
```

---

## 9. Common mistakes

| Mistake | Fix |
|---------|-----|
| Hardcoded namespace in templates | Use `{{ .Release.Namespace }}` |
| `latest` tag in prod | Pin image tag to git sha in CI |
| Giant single values.yaml | Split per environment |
| No resource limits | Always set requests + limits |
| Secrets in Git | Key Vault + External Secrets |

---

## Related

- [Helm Deep Dive](DevOps-20-Helm.md)
- [Helm Cheatsheet](../Cheatsheets/Helm-Cheatsheet.md)
- [SQL Server + Flyway](../Databases/SQL-Server-Azure.md)
- [PetClinic Phase 2](../../Projects/PetClinic.md)
