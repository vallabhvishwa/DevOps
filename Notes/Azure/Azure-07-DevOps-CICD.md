# Azure DevOps & CI/CD Services - Complete Guide
## Azure DevOps, Container Registry, Artifacts

---

# 1. AZURE DEVOPS

## What It Is
Comprehensive DevOps platform with repos, pipelines, boards, test plans, and artifacts.

## Services
```
AZURE DEVOPS SERVICES:
┌─────────────────────────────────────────────────────────────────┐
│ AZURE REPOS:        Git repositories                           │
│ AZURE PIPELINES:    CI/CD pipelines                            │
│ AZURE BOARDS:       Work tracking, Kanban, Scrum               │
│ AZURE TEST PLANS:   Test management                            │
│ AZURE ARTIFACTS:    Package management                         │
└─────────────────────────────────────────────────────────────────┘
```

## Azure Pipelines

### YAML Pipeline Structure
```yaml
# azure-pipelines.yml
trigger:
  - main
  - develop

pool:
  vmImage: 'ubuntu-latest'

variables:
  - name: environment
    value: 'production'
  - group: my-variable-group  # Link to variable group

stages:
  - stage: Build
    jobs:
      - job: BuildJob
        steps:
          - task: NodeTool@0
            inputs:
              versionSpec: '18.x'
          
          - script: |
              npm ci
              npm run build
            displayName: 'Build application'
          
          - task: PublishPipelineArtifact@1
            inputs:
              targetPath: '$(Build.ArtifactStagingDirectory)'
              artifact: 'drop'

  - stage: Deploy
    dependsOn: Build
    condition: succeeded()
    jobs:
      - deployment: DeployToAKS
        environment: 'production'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: KubernetesManifest@0
                  inputs:
                    action: 'deploy'
                    kubernetesServiceConnection: 'my-aks-connection'
                    manifests: '$(Pipeline.Workspace)/drop/manifests/*.yaml'
```

### Service Connections
```bash
# Create service connection via CLI (az devops extension)
az devops service-endpoint azurerm create \
  --azure-rm-service-principal-id <sp-id> \
  --azure-rm-subscription-id <sub-id> \
  --azure-rm-subscription-name "My Subscription" \
  --azure-rm-tenant-id <tenant-id> \
  --name "Azure Connection"

# Create Kubernetes connection
az devops service-endpoint kubernetes create \
  --kubernetes-url https://myaks.hcp.eastus.azmk8s.io:443 \
  --name "AKS Connection" \
  --authorization-type Kubeconfig
```

### Common Pipeline Tasks
```yaml
# Docker build and push
- task: Docker@2
  inputs:
    containerRegistry: 'myACRConnection'
    repository: 'myapp'
    command: 'buildAndPush'
    Dockerfile: '**/Dockerfile'
    tags: '$(Build.BuildId)'

# Kubernetes deploy
- task: KubernetesManifest@0
  inputs:
    action: 'deploy'
    kubernetesServiceConnection: 'myAKS'
    namespace: 'production'
    manifests: 'manifests/*.yaml'
    containers: 'myacr.azurecr.io/myapp:$(Build.BuildId)'

# Helm deploy
- task: HelmDeploy@0
  inputs:
    connectionType: 'Azure Resource Manager'
    azureSubscription: 'mySubscription'
    azureResourceGroup: 'myRG'
    kubernetesCluster: 'myAKS'
    command: 'upgrade'
    chartType: 'FilePath'
    chartPath: 'charts/myapp'
    releaseName: 'myapp'
    overrideValues: 'image.tag=$(Build.BuildId)'

# Azure CLI task
- task: AzureCLI@2
  inputs:
    azureSubscription: 'mySubscription'
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
      az aks get-credentials -g myRG -n myAKS
      kubectl get pods
```

---

# 2. AZURE CONTAINER REGISTRY (ACR)

## What It Is
Managed Docker registry for storing and managing container images and artifacts.

## Tiers
```
ACR TIERS:
┌──────────────┬────────────┬────────────────┬─────────────────────┐
│ Tier         │ Storage    │ Throughput     │ Features            │
├──────────────┼────────────┼────────────────┼─────────────────────┤
│ Basic        │ 10 GB      │ Low            │ Dev/test            │
│ Standard     │ 100 GB     │ Medium         │ Production          │
│ Premium      │ 500 GB     │ High           │ Geo-replication,    │
│              │            │                │ Private Link        │
└──────────────┴────────────┴────────────────┴─────────────────────┘
```

## CLI Commands

```bash
# Create ACR
az acr create \
  --resource-group myRG \
  --name myacr \
  --sku Premium \
  --admin-enabled false

# Login to ACR
az acr login --name myacr

# Build image in ACR (no local Docker needed)
az acr build \
  --registry myacr \
  --image myapp:v1 \
  --file Dockerfile .

# List repositories
az acr repository list --name myacr -o table

# List tags
az acr repository show-tags --name myacr --repository myapp -o table

# Delete image
az acr repository delete --name myacr --image myapp:v1

# Enable geo-replication
az acr replication create \
  --registry myacr \
  --location westus

# Enable content trust
az acr config content-trust update \
  --registry myacr \
  --status enabled

# Create private endpoint
az network private-endpoint create \
  --name acrPrivateEndpoint \
  --resource-group myRG \
  --vnet-name myVNet \
  --subnet PrivateEndpointSubnet \
  --private-connection-resource-id $(az acr show -n myacr --query id -o tsv) \
  --group-id registry \
  --connection-name acrConnection

# Enable admin account (not recommended for production)
az acr update --name myacr --admin-enabled true

# Get admin credentials
az acr credential show --name myacr

# Assign AKS pull permission
az aks update \
  --resource-group myRG \
  --name myAKS \
  --attach-acr myacr
```

## ACR Tasks (CI/CD Built-in)
```bash
# Quick build
az acr build --registry myacr --image myapp:latest .

# Create scheduled task
az acr task create \
  --registry myacr \
  --name buildTask \
  --image myapp:{{.Run.ID}} \
  --context https://github.com/user/repo.git \
  --file Dockerfile \
  --git-access-token $GIT_TOKEN \
  --schedule "0 21 * * *"  # Daily at 9 PM UTC

# Create trigger on base image update
az acr task create \
  --registry myacr \
  --name baseUpdate \
  --image myapp:{{.Run.ID}} \
  --context https://github.com/user/repo.git \
  --file Dockerfile \
  --git-access-token $GIT_TOKEN \
  --base-image-trigger-enabled true
```

---

# 3. AZURE ARTIFACTS

## What It Is
Package management for Maven, npm, NuGet, Python packages. Host private packages.

## Supported Feeds
```
PACKAGE TYPES:
┌─────────────────────────────────────────────────────────────────┐
│ npm:        Node.js packages                                   │
│ NuGet:      .NET packages                                      │
│ Maven:      Java packages                                      │
│ Python:     pip packages                                       │
│ Universal:  Any file type                                      │
└─────────────────────────────────────────────────────────────────┘
```

## Configuration

```bash
# npm - .npmrc
registry=https://pkgs.dev.azure.com/myorg/_packaging/myfeed/npm/registry/
always-auth=true

# Maven - pom.xml
<repository>
  <id>myfeed</id>
  <url>https://pkgs.dev.azure.com/myorg/_packaging/myfeed/maven/v1</url>
</repository>

# NuGet - nuget.config
<packageSources>
  <add key="myfeed" value="https://pkgs.dev.azure.com/myorg/_packaging/myfeed/nuget/v3/index.json" />
</packageSources>

# Python - pip.conf
[global]
index-url=https://pkgs.dev.azure.com/myorg/_packaging/myfeed/pypi/simple/
```

---

# CICD BEST PRACTICES

```
CI/CD CHECKLIST:
┌─────────────────────────────────────────────────────────────────┐
│ ✓ Use YAML pipelines (version controlled)                     │
│ ✓ Store secrets in Key Vault, not variables                   │
│ ✓ Use service connections with minimal permissions            │
│ ✓ Implement branch policies                                   │
│ ✓ Run security scans in pipeline                              │
│ ✓ Use deployment environments with approvals                  │
│ ✓ Implement blue-green or canary deployments                  │
│ ✓ Tag images with build ID, not just 'latest'                 │
│ ✓ Use multi-stage pipelines                                   │
│ ✓ Cache dependencies for faster builds                        │
└─────────────────────────────────────────────────────────────────┘
```
