# Terraform Cheatsheet

> See [Terraform Guide](../Terraform/DevOps-08-Terraform.md)

## Workflow

```bash
terraform init
terraform fmt -recursive
terraform validate
terraform plan -out=tfplan
terraform apply tfplan
terraform destroy
```

## State

```bash
terraform state list
terraform state show azurerm_resource_group.main
terraform state mv OLD NEW
terraform import azurerm_resource_group.main /subscriptions/.../resourceGroups/myRG
terraform workspace list
terraform workspace select dev
```

## Variables

```bash
terraform plan -var="environment=dev"
terraform plan -var-file=dev.tfvars
export TF_VAR_db_password="$(vault read -field=password secret/db)"
```

## Modules

```hcl
module "aks" {
  source = "./modules/aks"
  environment = var.environment
}
```

## Useful flags

```bash
terraform plan -target=azurerm_kubernetes_cluster.main
terraform apply -auto-approve          # CI only
terraform refresh
terraform output -json
```

## Azure auth

```bash
az login
export ARM_SUBSCRIPTION_ID="..."
export ARM_TENANT_ID="..."
export ARM_CLIENT_ID="..."        # SP
export ARM_CLIENT_SECRET="..."    # SP
# Or use Azure CLI auth: use_azuread_auth in provider
```

## .gitignore essentials

```
.terraform/
*.tfstate
*.tfstate.*
*.tfvars
!example.tfvars
.terraform.lock.hcl
```
