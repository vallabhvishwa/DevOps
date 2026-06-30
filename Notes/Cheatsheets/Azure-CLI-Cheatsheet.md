# Azure CLI Cheatsheet

> See [Azure Index](../Azure/Azure-00-INDEX.md) | [Private Link](../Azure/Azure-Private-Link-KeyVault-CICD.md)

## Auth & context

```bash
az login
az account list -o table
az account set --subscription "My Subscription"
az account show
```

## Resource groups

```bash
az group list -o table
az group create -n myRG -l eastus2
az group delete -n myRG --yes
```

## AKS

```bash
az aks list -g myRG -o table
az aks get-credentials -g myRG -n myAKS --admin
az aks show -g myRG -n myAKS
az aks nodepool list --cluster-name myAKS -g myRG
az aks get-versions -l eastus2
```

## ACR

```bash
az acr list -o table
az acr login --name myregistry
az acr repository list --name myregistry
az acr repository show-tags --name myregistry --repository myapp
```

## Key Vault

```bash
az keyvault list -o table
az keyvault secret set --vault-name myvault -n sql-password --value "..."
az keyvault secret show --vault-name myvault -n sql-password --query value -o tsv
```

## SQL

```bash
az sql server list -g myRG -o table
az sql db list --server myserver -g myRG -o table
az sql db show-connection-string --client ado.net -s myserver -n mydb
```

## Networking

```bash
az network vnet list -g myRG -o table
az network private-endpoint list -g myRG -o table
az network private-dns zone list -g myRG -o table
```

## Monitor

```bash
az monitor activity-log list -g myRG --offset 1h
az monitor metrics list --resource $AKS_ID --metric "node_cpu_usage_percentage"
```

## VMs

```bash
az vm list -g myRG -d -o table
az vm start -g myRG -n myVM
az vm stop -g myRG -n myVM
```
