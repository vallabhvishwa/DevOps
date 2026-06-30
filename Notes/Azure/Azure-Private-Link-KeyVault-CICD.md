# Azure Private Link & Key Vault in CI/CD

> **last_reviewed:** 2026-06-30  
> **See also:** [Azure Networking](../Azure/Azure-02-Networking-Services.md) | [Azure Identity](../Azure/Azure-05-Identity-Security.md) | [Security](../Security/DevOps-13-Security.md)

---

## 1. Why Private Link

Public endpoints expose Azure PaaS to the internet. Enterprise AKS/Jenkins environments use **Private Endpoints** so traffic stays on the Microsoft backbone.

```
Jenkins/AKS ──▶ Private Endpoint ──▶ Key Vault / SQL / ACR / Storage
                     (10.x IP)
                     privatelink.* DNS zone
```

**Common privatelink zones:**

| Service | DNS zone |
|---------|----------|
| Key Vault | `privatelink.vaultcore.azure.net` |
| Azure SQL | `privatelink.database.windows.net` |
| ACR | `privatelink.azurecr.io` |
| Blob Storage | `privatelink.blob.core.windows.net` |
| AKS API | `privatelink.<region>.azmk8s.io` |

---

## 2. Private DNS zone

Private endpoint gets a private IP; DNS zone maps `myvault.vault.azure.net` → `10.0.1.4`.

```hcl
resource "azurerm_private_dns_zone" "kv" {
  name                = "privatelink.vaultcore.azure.net"
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_private_dns_zone_virtual_network_link" "kv" {
  name                  = "kv-vnet-link"
  resource_group_name   = azurerm_resource_group.main.name
  private_dns_zone_name = azurerm_private_dns_zone.kv.name
  virtual_network_id    = azurerm_virtual_network.main.id
}
```

**Verify from AKS pod:**

```bash
nslookup myvault.vault.azure.net
# Should return 10.x private IP, not public
```

---

## 3. Key Vault access patterns

| Pattern | Use when |
|---------|----------|
| **Managed Identity** | AKS workloads, Jenkins on Azure VM |
| **Service Principal** | Jenkins with stored credentials |
| **Workload Identity** | Modern AKS — federated credential |

### AKS — Secrets Store CSI Driver

```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: azure-kv
spec:
  provider: azure
  parameters:
    usePodIdentity: "false"
    useVMManagedIdentity: "true"
    userAssignedIdentityID: "<client-id>"
    keyvaultName: "myvault"
    objects: |
      array:
        - |
          objectName: sql-password
          objectType: secret
    tenantId: "<tenant-id>"
```

### Jenkins — Azure Key Vault plugin

```groovy
def secrets = [
    [path: 'kv/acr-credentials', secretValues: [
        [vaultKey: 'password', envVar: 'ACR_PASSWORD']
    ]]
]
wrap([$class: 'VaultBuildWrapper', vaultSecrets: secrets]) {
    sh 'docker login ...'
}
```

---

## 4. HTTP_PROXY and NO_PROXY

Corporate Jenkins/agents behind Squid proxy must bypass private Azure endpoints:

```groovy
environment {
    HTTPS_PROXY = 'http://squid.corp.com:3128'
    HTTP_PROXY  = 'http://squid.corp.com:3128'
    NO_PROXY    = '''
        127.0.0.1,localhost,
        .vault.azure.net,.privatelink.vaultcore.azure.net,
        .database.windows.net,.privatelink.database.windows.net,
        .azurecr.io,.privatelink.azurecr.io,
        .blob.core.windows.net,.privatelink.blob.core.windows.net,
        .azmk8s.io,.privatelink.eastus2.azmk8s.io
    '''.replaceAll(/\s/, '')
}
```

**Symptom when misconfigured:** Terraform/Azure CLI hangs or SSL errors reaching Key Vault or ACR despite private endpoints being correct.

---

## 5. ACR with Private Link

```bash
# AKS must use private DNS for ACR
# az acr login works from agent with correct DNS + NO_PROXY

az acr login --name myregistry
docker pull myregistry.azurecr.io/myapp:latest
```

**AKS integration:**

```hcl
resource "azurerm_role_assignment" "aks_acr" {
  scope                = azurerm_container_registry.main.id
  role_definition_name = "AcrPull"
  principal_id         = azurerm_kubernetes_cluster.main.kubelet_identity[0].object_id
}
```

---

## 6. CI/CD secret flow (recommended)

```
Developer ──▶ Jenkins (no secrets in Git)
                  │
                  ▼
            Key Vault (source of truth)
                  │
        ┌─────────┴─────────┐
        ▼                   ▼
   Terraform            Helm deploy
   (ARM_SP from KV)     (CSI / ExternalSecrets)
```

**Never:**
- Commit secrets to Git
- Put prod passwords in `values-prod.yaml`
- Log credential env vars in pipeline output

---

## 7. Troubleshooting checklist

| Issue | Check |
|-------|-------|
| 403 Forbidden to Key Vault | RBAC: `Key Vault Secrets User` on MI |
| DNS resolves to public IP | Private DNS zone not linked to VNet |
| Works from VM, not AKS | AKS in different VNet — peer or link DNS |
| Jenkins timeout | `NO_PROXY` missing privatelink suffix |
| ACR ImagePullBackOff | `AcrPull` role + correct DNS |

---

## Related

- [SQL Server Azure](../Databases/SQL-Server-Azure.md)
- [Jenkins Groovy](Jenkins-Groovy-Shared-Libraries.md)
- [Azure Troubleshooting](../../Troubleshooting/Azure/09-Azure-Cloud-Troubleshooting.md)
- [Networking Troubleshooting](../../Troubleshooting/Networking/04-Networking-Troubleshooting.md)
