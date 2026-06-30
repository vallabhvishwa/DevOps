# Helm Cheatsheet

> See [Helm Deep Dive](../Helm/DevOps-20-Helm.md) | [Enterprise Patterns](../Helm/Helm-Enterprise-Patterns.md)

## Repo & search

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo nginx
```

## Install / upgrade

```bash
helm install RELEASE ./chart -n NS --create-namespace
helm upgrade --install RELEASE ./chart -f values-dev.yaml -n NS
helm upgrade RELEASE ./chart --set replicaCount=3
helm uninstall RELEASE -n NS
```

## Debug (use before every deploy)

```bash
helm lint ./chart -f values-dev.yaml
helm template RELEASE ./chart -f values-dev.yaml
helm template RELEASE ./chart --debug 2>&1 | less
helm get manifest RELEASE -n NS
helm get values RELEASE -n NS
```

## History & rollback

```bash
helm list -n NS
helm history RELEASE -n NS
helm rollback RELEASE 2 -n NS
```

## Dependencies & package

```bash
helm dependency update ./chart
helm package ./chart
helm push mychart-1.0.0.tgz oci://myregistry.azurecr.io/helm
```

## Diff (plugin)

```bash
helm plugin install https://github.com/databus23/helm-diff
helm diff upgrade RELEASE ./chart -f values-prod.yaml
```

## Common template functions

```yaml
{{ .Values.replicaCount }}
{{ .Values.image.tag | default .Chart.AppVersion }}
{{ include "mychart.fullname" . }}
{{- if .Values.ingress.enabled }}
{{- range $k, $v := .Values.env }}
```
