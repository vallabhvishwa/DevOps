# Azure Identity & Security - Complete Guide
## Entra ID, Key Vault, RBAC, Managed Identities, Defender

---

# 1. MICROSOFT ENTRA ID (Azure AD)

## What It Is
Cloud-based identity and access management service. Provides authentication and authorization for Azure resources and thousands of SaaS apps.

## Key Concepts
```
CORE CONCEPTS:
┌─────────────────────────────────────────────────────────────────┐
│ TENANT:           Directory instance (organization)            │
│ USER:             Person with identity                         │
│ GROUP:            Collection of users                          │
│ SERVICE PRINCIPAL: Identity for applications                   │
│ MANAGED IDENTITY:  Azure-managed identity for resources        │
│ APP REGISTRATION:  Application definition in Entra ID          │
└─────────────────────────────────────────────────────────────────┘
```

## CLI Commands

```bash
# List users
az ad user list --output table

# Create user
az ad user create \
  --display-name "John Doe" \
  --user-principal-name john@contoso.com \
  --password "TempP@ss123!"

# List groups
az ad group list --output table

# Create group
az ad group create \
  --display-name "Developers" \
  --mail-nickname "developers"

# Add user to group
az ad group member add \
  --group "Developers" \
  --member-id <user-object-id>

# Create app registration
az ad app create --display-name "MyApp"

# Create service principal
az ad sp create-for-rbac --name "MyServicePrincipal"
```

---

# 2. MANAGED IDENTITIES

## What It Is
Azure automatically managed identities for authenticating to Azure services. No credentials in code.

## Types
```
MANAGED IDENTITY TYPES:
┌─────────────────────────────────────────────────────────────────┐
│ SYSTEM-ASSIGNED:                                                │
│ - Created with resource                                        │
│ - Deleted with resource                                        │
│ - One-to-one relationship                                      │
│ - Simpler lifecycle                                            │
│                                                                 │
│ USER-ASSIGNED:                                                  │
│ - Created separately                                           │
│ - Can be shared across resources                               │
│ - Independent lifecycle                                        │
│ - More flexible                                                │
└─────────────────────────────────────────────────────────────────┘
```

## CLI Commands

```bash
# Enable system-assigned identity on VM
az vm identity assign \
  --resource-group myRG \
  --name myVM

# Create user-assigned identity
az identity create \
  --resource-group myRG \
  --name myIdentity

# Assign user-assigned identity to VM
az vm identity assign \
  --resource-group myRG \
  --name myVM \
  --identities myIdentity

# Get identity principal ID
az vm identity show \
  --resource-group myRG \
  --name myVM \
  --query principalId -o tsv

# Assign role to managed identity
az role assignment create \
  --assignee <principal-id> \
  --role "Storage Blob Data Reader" \
  --scope /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Storage/storageAccounts/<account>
```

## Using in Code
```python
# Python example - no credentials needed!
from azure.identity import DefaultAzureCredential
from azure.storage.blob import BlobServiceClient

credential = DefaultAzureCredential()
blob_service = BlobServiceClient(
    account_url="https://mystorageaccount.blob.core.windows.net",
    credential=credential
)
```

---

# 3. AZURE KEY VAULT

## What It Is
Centralized secrets management. Store and access secrets, keys, and certificates securely.

## What It Stores
```
KEY VAULT OBJECTS:
┌─────────────────────────────────────────────────────────────────┐
│ SECRETS:          Passwords, connection strings, API keys      │
│ KEYS:             Cryptographic keys for encryption            │
│ CERTIFICATES:     SSL/TLS certificates                         │
└─────────────────────────────────────────────────────────────────┘
```

## CLI Commands

```bash
# Create Key Vault
az keyvault create \
  --name mykeyvault \
  --resource-group myRG \
  --location eastus \
  --enable-rbac-authorization true

# Set secret
az keyvault secret set \
  --vault-name mykeyvault \
  --name "DatabasePassword" \
  --value "MySecretP@ss!"

# Get secret
az keyvault secret show \
  --vault-name mykeyvault \
  --name "DatabasePassword" \
  --query value -o tsv

# List secrets
az keyvault secret list \
  --vault-name mykeyvault \
  --output table

# Create key
az keyvault key create \
  --vault-name mykeyvault \
  --name "MyKey" \
  --kty RSA \
  --size 2048

# Import certificate
az keyvault certificate import \
  --vault-name mykeyvault \
  --name "MyCert" \
  --file ./certificate.pfx \
  --password "certpassword"

# Grant access (RBAC)
az role assignment create \
  --role "Key Vault Secrets User" \
  --assignee <principal-id> \
  --scope /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.KeyVault/vaults/mykeyvault

# Enable soft delete recovery
az keyvault recover --name mykeyvault

# Enable private endpoint
az network private-endpoint create \
  --name kvPrivateEndpoint \
  --resource-group myRG \
  --vnet-name myVNet \
  --subnet PrivateEndpointSubnet \
  --private-connection-resource-id $(az keyvault show -n mykeyvault --query id -o tsv) \
  --group-id vault \
  --connection-name kvConnection
```

---

# 4. AZURE RBAC (Role-Based Access Control)

## What It Is
Authorization system to manage access to Azure resources. Who (principal) can do what (role) on what scope.

## Key Concepts
```
RBAC COMPONENTS:
┌─────────────────────────────────────────────────────────────────┐
│ SECURITY PRINCIPAL:  User, group, service principal, managed ID │
│ ROLE DEFINITION:     Collection of permissions                  │
│ SCOPE:               Level where access applies                 │
│ ASSIGNMENT:          Principal + Role + Scope                   │
│                                                                 │
│ SCOPE HIERARCHY:                                                 │
│ Management Group → Subscription → Resource Group → Resource     │
└─────────────────────────────────────────────────────────────────┘
```

## Built-in Roles
```
COMMON BUILT-IN ROLES:
┌───────────────────────────────┬─────────────────────────────────┐
│ Role                          │ Description                     │
├───────────────────────────────┼─────────────────────────────────┤
│ Owner                         │ Full access + assign roles      │
│ Contributor                   │ Full access, no role assignment │
│ Reader                        │ View only                       │
│ User Access Administrator     │ Manage user access              │
│ Storage Blob Data Contributor │ Read/write blob data            │
│ Storage Blob Data Reader      │ Read blob data                  │
│ Key Vault Secrets User        │ Read secrets                    │
│ AKS Cluster Admin             │ Full AKS admin                  │
│ Virtual Machine Contributor   │ Manage VMs                      │
└───────────────────────────────┴─────────────────────────────────┘
```

## CLI Commands

```bash
# List role definitions
az role definition list --output table

# List role assignments
az role assignment list --output table

# Assign role to user
az role assignment create \
  --assignee user@contoso.com \
  --role "Contributor" \
  --resource-group myRG

# Assign role to service principal
az role assignment create \
  --assignee <app-id> \
  --role "Storage Blob Data Contributor" \
  --scope /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Storage/storageAccounts/<account>

# Remove role assignment
az role assignment delete \
  --assignee user@contoso.com \
  --role "Contributor" \
  --resource-group myRG

# Create custom role
az role definition create --role-definition '{
  "Name": "Custom VM Operator",
  "Description": "Can start and stop VMs",
  "Actions": [
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/powerOff/action",
    "Microsoft.Compute/virtualMachines/read"
  ],
  "AssignableScopes": ["/subscriptions/<sub-id>"]
}'
```

---

# 5. AZURE POLICY

## What It Is
Governance service to enforce organizational standards and assess compliance at scale.

## Key Concepts
```
POLICY COMPONENTS:
┌─────────────────────────────────────────────────────────────────┐
│ POLICY DEFINITION:  What to evaluate and what action to take   │
│ POLICY ASSIGNMENT:  Application of policy to scope             │
│ INITIATIVE:         Collection of policies (policy set)        │
│ COMPLIANCE:         Assessment of resources against policies   │
│                                                                 │
│ EFFECTS:                                                        │
│ - Deny: Block non-compliant resources                          │
│ - Audit: Log non-compliance                                    │
│ - Append: Add fields to resources                              │
│ - Modify: Add/update/remove tags                               │
│ - DeployIfNotExists: Deploy if missing                         │
└─────────────────────────────────────────────────────────────────┘
```

## CLI Commands

```bash
# List policy definitions
az policy definition list --query "[].displayName" -o table

# Assign built-in policy
az policy assignment create \
  --name "RequireTag" \
  --display-name "Require Environment Tag" \
  --policy "/providers/Microsoft.Authorization/policyDefinitions/96670d01-0a4d-4649-9c89-2d3abc0a5025" \
  --scope /subscriptions/<sub>/resourceGroups/<rg> \
  --params '{"tagName": {"value": "Environment"}}'

# Check compliance
az policy state list \
  --resource-group myRG \
  --query "[?complianceState=='NonCompliant'].{Resource:resourceId,Policy:policyDefinitionName}"

# Create custom policy
az policy definition create \
  --name "AllowedLocations" \
  --display-name "Allowed Locations" \
  --mode Indexed \
  --rules '{
    "if": {
      "not": {
        "field": "location",
        "in": ["eastus", "westus"]
      }
    },
    "then": {
      "effect": "deny"
    }
  }'
```

---

# 6. MICROSOFT DEFENDER FOR CLOUD

## What It Is
Cloud security posture management (CSPM) and workload protection platform. Provides security recommendations and threat protection.

## Key Features
```
CAPABILITIES:
┌─────────────────────────────────────────────────────────────────┐
│ SECURE SCORE:       Overall security posture metric            │
│ RECOMMENDATIONS:    Prioritized security fixes                 │
│ REGULATORY COMPLIANCE: Track against standards                  │
│ WORKLOAD PROTECTION: Defender plans for specific workloads     │
│ ATTACK PATH ANALYSIS: Identify potential attack vectors        │
└─────────────────────────────────────────────────────────────────┘
```

## Defender Plans
```
DEFENDER PLANS:
- Defender for Servers
- Defender for Containers
- Defender for App Service
- Defender for Storage
- Defender for SQL
- Defender for Key Vault
- Defender for DNS
- Defender for Kubernetes
```

## CLI Commands

```bash
# Check security score
az security secure-score-controls list --output table

# Get recommendations
az security assessment list --output table

# Enable Defender plan
az security pricing create \
  --name VirtualMachines \
  --tier Standard

# Get alerts
az security alert list --output table
```

---

# SECURITY BEST PRACTICES

```
SECURITY CHECKLIST:
┌─────────────────────────────────────────────────────────────────┐
│ ✓ Use Managed Identities (no credentials in code)             │
│ ✓ Store secrets in Key Vault                                  │
│ ✓ Enable MFA for all users                                    │
│ ✓ Use RBAC with least privilege                               │
│ ✓ Enable Defender for Cloud                                   │
│ ✓ Use Private Endpoints for PaaS services                     │
│ ✓ Enable Azure Policy for governance                          │
│ ✓ Implement network segmentation                              │
│ ✓ Enable diagnostic logging                                   │
│ ✓ Regular access reviews                                      │
└─────────────────────────────────────────────────────────────────┘
```
