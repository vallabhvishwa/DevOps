# DevOps Engineer's Complete Reference Guide
# Part 8: Terraform & Infrastructure as Code

---

## Table of Contents

1. [Introduction to Infrastructure as Code](#1-introduction-to-infrastructure-as-code)
2. [Terraform Fundamentals](#2-terraform-fundamentals)
3. [HashiCorp Configuration Language (HCL)](#3-hashicorp-configuration-language-hcl)
4. [Terraform Core Concepts](#4-terraform-core-concepts)
5. [State Management](#5-state-management)
6. [Modules](#6-modules)
7. [Workspaces](#7-workspaces)
8. [Providers](#8-providers)
9. [Azure Provider Deep Dive](#9-azure-provider-deep-dive)
10. [Advanced HCL Patterns](#10-advanced-hcl-patterns)
11. [Testing Infrastructure Code](#11-testing-infrastructure-code)
12. [CI/CD for Terraform](#12-cicd-for-terraform)
13. [Policy as Code](#13-policy-as-code)
14. [Best Practices](#14-best-practices)
15. [Troubleshooting](#15-troubleshooting)

---

## 1. Introduction to Infrastructure as Code

### 1.1 What is Infrastructure as Code?

**Infrastructure as Code (IaC)** is the practice of managing and provisioning infrastructure through machine-readable configuration files rather than manual processes or interactive configuration tools.

```
Traditional Infrastructure Management:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   Admin logs into portal → Clicks through UI → Creates VM      │
│                                                                 │
│   Problems:                                                     │
│   ├── Not reproducible                                          │
│   ├── No version control                                        │
│   ├── Prone to human error                                      │
│   ├── No audit trail                                            │
│   ├── Difficult to scale                                        │
│   └── Environment drift                                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

Infrastructure as Code:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   Write code → Version control → Review → Apply automatically  │
│                                                                 │
│   Benefits:                                                     │
│   ├── Reproducible                                              │
│   ├── Version controlled                                        │
│   ├── Code review process                                       │
│   ├── Full audit trail                                          │
│   ├── Scalable                                                  │
│   ├── Self-documenting                                          │
│   └── Consistent environments                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Types of IaC Tools

```
IaC Tool Categories:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Declarative (Desired State):                                   │
│  ├── Terraform         - Multi-cloud, HCL                       │
│  ├── CloudFormation    - AWS native, JSON/YAML                  │
│  ├── ARM Templates     - Azure native, JSON                     │
│  ├── Bicep             - Azure native, DSL                      │
│  ├── Pulumi            - Multi-cloud, Programming languages     │
│  └── Crossplane        - Kubernetes-native                      │
│                                                                 │
│  Imperative (Procedural):                                       │
│  ├── Ansible           - Configuration management               │
│  ├── Chef              - Configuration management               │
│  ├── Puppet            - Configuration management               │
│  └── Scripts (Bash/PowerShell/Python)                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.3 Declarative vs Imperative

**Declarative Approach:**
You describe WHAT you want, the tool figures out HOW to achieve it.

```hcl
# Declarative: "I want 3 VMs"
resource "azurerm_virtual_machine" "example" {
  count = 3
  name  = "vm-${count.index}"
  # ... configuration
}
```

The tool will:
- Create 3 VMs if none exist
- Do nothing if 3 already exist
- Create 1 more if only 2 exist
- Delete 1 if 4 exist

**Imperative Approach:**
You describe HOW to achieve the desired state step by step.

```bash
# Imperative: "Create VMs one by one"
for i in 1 2 3; do
  az vm create --name "vm-$i" --resource-group myRG --image Ubuntu
done
```

The tool will:
- Always try to create 3 VMs
- May fail if they already exist
- Doesn't know current state

### 1.4 Why Terraform?

```
Terraform Advantages:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. Multi-Cloud Support                                         │
│     ├── AWS, Azure, GCP, and 3000+ providers                    │
│     └── Single tool for hybrid/multi-cloud                      │
│                                                                 │
│  2. Declarative Language (HCL)                                  │
│     ├── Human-readable                                          │
│     ├── Easy to learn                                           │
│     └── Purpose-built for infrastructure                        │
│                                                                 │
│  3. State Management                                            │
│     ├── Tracks real-world resources                             │
│     ├── Detects drift                                           │
│     └── Enables planning before applying                        │
│                                                                 │
│  4. Mature Ecosystem                                            │
│     ├── Large community                                         │
│     ├── Extensive documentation                                 │
│     ├── Terraform Registry (modules)                            │
│     └── Enterprise features available                           │
│                                                                 │
│  5. Plan Before Apply                                           │
│     ├── Preview changes before making them                      │
│     └── Prevents accidental destruction                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Terraform Fundamentals

### 2.1 Terraform Architecture

```
Terraform Architecture:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│                    ┌─────────────────┐                          │
│                    │  Configuration  │                          │
│                    │   Files (.tf)   │                          │
│                    └────────┬────────┘                          │
│                             │                                   │
│                             ▼                                   │
│                    ┌─────────────────┐                          │
│                    │  Terraform Core │                          │
│                    │   (terraform)   │                          │
│                    └────────┬────────┘                          │
│                             │                                   │
│              ┌──────────────┼──────────────┐                    │
│              │              │              │                    │
│              ▼              ▼              ▼                    │
│       ┌──────────┐   ┌──────────┐   ┌──────────┐                │
│       │  Azure   │   │   AWS    │   │   GCP    │                │
│       │ Provider │   │ Provider │   │ Provider │                │
│       └────┬─────┘   └────┬─────┘   └────┬─────┘                │
│            │              │              │                      │
│            ▼              ▼              ▼                      │
│       ┌──────────┐   ┌──────────┐   ┌──────────┐                │
│       │  Azure   │   │   AWS    │   │   GCP    │                │
│       │   APIs   │   │   APIs   │   │   APIs   │                │
│       └──────────┘   └──────────┘   └──────────┘                │
│                                                                 │
│                    ┌─────────────────┐                          │
│                    │   State File    │                          │
│                    │ (terraform.tfstate)                        │
│                    └─────────────────┘                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Components:**

1. **Configuration Files (.tf):** Your infrastructure code written in HCL
2. **Terraform Core:** The binary that processes configurations and manages lifecycle
3. **Providers:** Plugins that interact with specific APIs (Azure, AWS, Kubernetes, etc.)
4. **State File:** JSON file tracking what resources Terraform manages

### 2.2 Installation

**Windows (using Chocolatey):**
```powershell
choco install terraform
```

**Windows (Manual):**
```powershell
# Download from https://www.terraform.io/downloads
# Extract to C:\terraform
# Add C:\terraform to PATH

# Verify installation
terraform version
```

**Linux:**
```bash
# Ubuntu/Debian
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# Verify
terraform version
```

### 2.3 Terraform Workflow

```
Terraform Core Workflow:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   Write ──▶ Init ──▶ Plan ──▶ Apply ──▶ Destroy                │
│                                                                 │
│   1. WRITE                                                      │
│      └── Create .tf configuration files                         │
│                                                                 │
│   2. INIT (terraform init)                                      │
│      ├── Downloads providers                                    │
│      ├── Initializes backend                                    │
│      └── Downloads modules                                      │
│                                                                 │
│   3. PLAN (terraform plan)                                      │
│      ├── Reads current state                                    │
│      ├── Compares to configuration                              │
│      └── Shows what will change                                 │
│                                                                 │
│   4. APPLY (terraform apply)                                    │
│      ├── Executes the plan                                      │
│      ├── Creates/updates/deletes resources                      │
│      └── Updates state file                                     │
│                                                                 │
│   5. DESTROY (terraform destroy)                                │
│      └── Removes all managed resources                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2.4 First Terraform Configuration

Create a directory and file:

```
my-first-terraform/
├── main.tf
├── variables.tf
├── outputs.tf
└── terraform.tfvars
```

**main.tf:**
```hcl
# Configure the Azure Provider
terraform {
  required_version = ">= 1.0.0"
  
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

# Configure the provider
provider "azurerm" {
  features {}
}

# Create a Resource Group
resource "azurerm_resource_group" "example" {
  name     = var.resource_group_name
  location = var.location

  tags = {
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}

# Create a Virtual Network
resource "azurerm_virtual_network" "example" {
  name                = "${var.prefix}-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name

  tags = azurerm_resource_group.example.tags
}

# Create a Subnet
resource "azurerm_subnet" "example" {
  name                 = "${var.prefix}-subnet"
  resource_group_name  = azurerm_resource_group.example.name
  virtual_network_name = azurerm_virtual_network.example.name
  address_prefixes     = ["10.0.1.0/24"]
}
```

**variables.tf:**
```hcl
variable "resource_group_name" {
  description = "Name of the resource group"
  type        = string
}

variable "location" {
  description = "Azure region for resources"
  type        = string
  default     = "eastus"
}

variable "prefix" {
  description = "Prefix for resource names"
  type        = string
  default     = "demo"
}

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "dev"
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}
```

**outputs.tf:**
```hcl
output "resource_group_id" {
  description = "The ID of the resource group"
  value       = azurerm_resource_group.example.id
}

output "vnet_id" {
  description = "The ID of the virtual network"
  value       = azurerm_virtual_network.example.id
}

output "subnet_id" {
  description = "The ID of the subnet"
  value       = azurerm_subnet.example.id
}
```

**terraform.tfvars:**
```hcl
resource_group_name = "my-terraform-rg"
location            = "eastus"
prefix              = "demo"
environment         = "dev"
```

### 2.5 Running Terraform

```bash
# Step 1: Initialize the working directory
terraform init

# Output:
# Initializing the backend...
# Initializing provider plugins...
# - Finding hashicorp/azurerm versions matching "~> 3.0"...
# - Installing hashicorp/azurerm v3.x.x...
# Terraform has been successfully initialized!

# Step 2: Validate the configuration
terraform validate

# Output:
# Success! The configuration is valid.

# Step 3: Format the code (auto-format)
terraform fmt

# Step 4: Plan the changes
terraform plan

# Output shows:
# + create    (new resource)
# ~ update    (modify in place)
# -/+ replace (destroy and recreate)
# - destroy   (delete)

# Step 5: Apply the changes
terraform apply

# Type 'yes' when prompted, or use:
terraform apply -auto-approve

# Step 6: Show current state
terraform show

# Step 7: List resources in state
terraform state list

# Step 8: Destroy all resources
terraform destroy
```

---

## 3. HashiCorp Configuration Language (HCL)

### 3.1 HCL Syntax Basics

HCL (HashiCorp Configuration Language) is a declarative language designed for defining infrastructure.

**Basic Structure:**

```hcl
# Comments start with #
// This is also a comment
/* 
   Multi-line
   comment
*/

# Block syntax
block_type "label1" "label2" {
  argument = "value"
  
  nested_block {
    nested_argument = "nested_value"
  }
}

# Examples of block types:
# - resource
# - data
# - variable
# - output
# - provider
# - terraform
# - module
# - locals
```

### 3.2 Data Types

```hcl
# String
name = "my-resource"
multiline = <<-EOT
  This is a
  multi-line string
  using heredoc syntax
EOT

# Number
count = 3
percentage = 99.9

# Boolean
enabled = true
disabled = false

# List (Array)
availability_zones = ["us-east-1a", "us-east-1b", "us-east-1c"]
mixed_list = ["string", 123, true]

# Map (Object/Dictionary)
tags = {
  Environment = "production"
  Team        = "platform"
  CostCenter  = "12345"
}

# Tuple (Fixed-length list with specific types)
# Mainly used in type constraints
tuple_example = ["string", 123, true]

# Object (Structured type with named attributes)
# Mainly used in type constraints
object_example = {
  name    = "example"
  enabled = true
  count   = 5
}

# Null
optional_value = null
```

### 3.3 Variables

**Variable Declaration:**

```hcl
# Basic variable
variable "instance_count" {
  description = "Number of instances to create"
  type        = number
  default     = 1
}

# String variable with validation
variable "environment" {
  description = "Deployment environment"
  type        = string
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

# List variable
variable "allowed_ips" {
  description = "List of allowed IP addresses"
  type        = list(string)
  default     = []
}

# Map variable
variable "instance_sizes" {
  description = "VM sizes per environment"
  type        = map(string)
  default = {
    dev     = "Standard_B1s"
    staging = "Standard_B2s"
    prod    = "Standard_D4s_v3"
  }
}

# Complex object variable
variable "database_config" {
  description = "Database configuration"
  type = object({
    name            = string
    sku             = string
    storage_mb      = number
    backup_retention = optional(number, 7)
    geo_redundant   = optional(bool, false)
  })
  
  default = {
    name       = "mydb"
    sku        = "GP_Gen5_2"
    storage_mb = 5120
  }
}

# Sensitive variable (won't be shown in logs)
variable "database_password" {
  description = "Database admin password"
  type        = string
  sensitive   = true
}

# Variable without default (must be provided)
variable "subscription_id" {
  description = "Azure subscription ID"
  type        = string
}
```

**Providing Variable Values:**

```bash
# 1. terraform.tfvars (auto-loaded)
# terraform.tfvars
instance_count = 3
environment    = "prod"

# 2. *.auto.tfvars (auto-loaded)
# prod.auto.tfvars
environment = "prod"

# 3. Command line
terraform apply -var="instance_count=5" -var="environment=staging"

# 4. Variable file
terraform apply -var-file="production.tfvars"

# 5. Environment variables
export TF_VAR_instance_count=3
export TF_VAR_environment=prod
terraform apply
```

**Variable Precedence (lowest to highest):**
1. Default value in variable declaration
2. Environment variables (TF_VAR_*)
3. terraform.tfvars
4. *.auto.tfvars (alphabetical order)
5. -var-file command line option
6. -var command line option

### 3.4 Local Values

Local values are like constants - computed once and reusable:

```hcl
locals {
  # Simple value
  project_name = "myproject"
  
  # Computed value
  resource_prefix = "${var.environment}-${local.project_name}"
  
  # Complex computed value
  common_tags = {
    Project     = local.project_name
    Environment = var.environment
    ManagedBy   = "Terraform"
    CreatedAt   = timestamp()
  }
  
  # Conditional value
  is_production = var.environment == "prod"
  
  # List transformation
  uppercase_zones = [for z in var.availability_zones : upper(z)]
  
  # Map from list
  zone_map = { for idx, zone in var.availability_zones : "zone${idx}" => zone }
}

# Usage
resource "azurerm_resource_group" "example" {
  name     = "${local.resource_prefix}-rg"
  location = var.location
  tags     = local.common_tags
}
```

### 3.5 Outputs

```hcl
# Basic output
output "resource_group_name" {
  description = "The name of the resource group"
  value       = azurerm_resource_group.example.name
}

# Sensitive output (hidden in CLI output)
output "database_connection_string" {
  description = "Database connection string"
  value       = azurerm_postgresql_server.example.connection_string
  sensitive   = true
}

# Conditional output
output "load_balancer_ip" {
  description = "Load balancer public IP"
  value       = var.create_lb ? azurerm_public_ip.lb[0].ip_address : null
}

# Complex output
output "vm_details" {
  description = "Details of created VMs"
  value = {
    for vm in azurerm_virtual_machine.example :
    vm.name => {
      id         = vm.id
      private_ip = vm.private_ip_address
      location   = vm.location
    }
  }
}
```

### 3.6 Expressions and Functions

**String Functions:**

```hcl
locals {
  # String manipulation
  upper_name   = upper("hello")              # "HELLO"
  lower_name   = lower("HELLO")              # "hello"
  title_name   = title("hello world")        # "Hello World"
  trimmed      = trim("  hello  ", " ")      # "hello"
  replaced     = replace("hello", "l", "L")  # "heLLo"
  
  # String formatting
  formatted = format("Hello, %s!", var.name)        # "Hello, John!"
  padded    = format("%05d", 42)                    # "00042"
  
  # Joining and splitting
  joined = join(", ", ["a", "b", "c"])              # "a, b, c"
  split  = split(",", "a,b,c")                      # ["a", "b", "c"]
  
  # Substrings
  substring = substr("hello", 0, 3)                 # "hel"
  
  # Regex
  matched = regex("^[a-z]+", "hello123")            # "hello"
  matches = regexall("[0-9]+", "a1b2c3")            # ["1", "2", "3"]
}
```

**Collection Functions:**

```hcl
locals {
  my_list = ["a", "b", "c"]
  my_map  = { a = 1, b = 2, c = 3 }
  
  # Length
  list_len = length(local.my_list)                  # 3
  map_len  = length(local.my_map)                   # 3
  
  # Element access
  first = element(local.my_list, 0)                 # "a"
  value = lookup(local.my_map, "a", "default")      # 1
  
  # Keys and values
  keys   = keys(local.my_map)                       # ["a", "b", "c"]
  values = values(local.my_map)                     # [1, 2, 3]
  
  # Merge maps
  merged = merge(
    { a = 1 },
    { b = 2 },
    { c = 3 }
  )                                                  # { a = 1, b = 2, c = 3 }
  
  # List operations
  concat_list = concat(["a"], ["b"], ["c"])         # ["a", "b", "c"]
  flat_list   = flatten([["a"], ["b", "c"]])        # ["a", "b", "c"]
  distinct    = distinct(["a", "b", "a"])           # ["a", "b"]
  sorted      = sort(["c", "a", "b"])               # ["a", "b", "c"]
  reversed    = reverse(["a", "b", "c"])            # ["c", "b", "a"]
  
  # Filtering
  compacted = compact(["a", "", "b", null, "c"])    # ["a", "b", "c"]
  
  # Contains
  has_a = contains(local.my_list, "a")              # true
}
```

**Numeric Functions:**

```hcl
locals {
  # Math
  abs_val   = abs(-5)                               # 5
  ceiling   = ceil(4.3)                             # 5
  floored   = floor(4.7)                            # 4
  max_val   = max(1, 5, 3)                          # 5
  min_val   = min(1, 5, 3)                          # 1
  power     = pow(2, 3)                             # 8
  sign_val  = signum(-5)                            # -1
  
  # Parsing
  parsed_int = parseint("42", 10)                   # 42
}
```

**Date/Time Functions:**

```hcl
locals {
  now          = timestamp()                         # "2024-01-15T10:30:00Z"
  formatted    = formatdate("YYYY-MM-DD", timestamp()) # "2024-01-15"
  future_date  = timeadd(timestamp(), "24h")        # 24 hours from now
}
```

**Encoding Functions:**

```hcl
locals {
  # JSON
  json_encoded = jsonencode({ name = "test", count = 5 })
  json_decoded = jsondecode("{\"name\":\"test\"}")
  
  # YAML
  yaml_encoded = yamlencode({ name = "test" })
  yaml_decoded = yamldecode("name: test")
  
  # Base64
  b64_encoded = base64encode("hello")               # "aGVsbG8="
  b64_decoded = base64decode("aGVsbG8=")            # "hello"
  
  # File encoding
  file_b64 = filebase64("./files/script.sh")
  
  # URL encoding
  url_encoded = urlencode("hello world")            # "hello%20world"
}
```

**Filesystem Functions:**

```hcl
locals {
  # Read file content
  script_content = file("${path.module}/scripts/init.sh")
  
  # Read and base64 encode
  encoded_script = filebase64("${path.module}/scripts/init.sh")
  
  # Check if file exists
  file_exists = fileexists("${path.module}/config.json")
  
  # Read directory
  config_files = fileset(path.module, "configs/*.json")
  
  # Template file
  rendered = templatefile("${path.module}/templates/config.tpl", {
    hostname = var.hostname
    port     = var.port
  })
  
  # Path references
  current_module = path.module     # Path to current module
  root_module    = path.root       # Path to root module
  current_dir    = path.cwd        # Current working directory
}
```

### 3.7 Conditional Expressions

```hcl
# Ternary operator: condition ? true_value : false_value
locals {
  instance_type = var.environment == "prod" ? "Standard_D4s_v3" : "Standard_B1s"
  
  # Nested conditions
  size = (
    var.environment == "prod" ? "large" :
    var.environment == "staging" ? "medium" :
    "small"
  )
}

# Conditional resource creation
resource "azurerm_public_ip" "example" {
  count = var.create_public_ip ? 1 : 0  # Create only if true
  
  name                = "pip-example"
  location            = var.location
  resource_group_name = var.resource_group_name
  allocation_method   = "Static"
}

# Reference conditional resource
resource "azurerm_network_interface" "example" {
  # ...
  
  ip_configuration {
    # Reference the conditional public IP
    public_ip_address_id = var.create_public_ip ? azurerm_public_ip.example[0].id : null
  }
}
```

### 3.8 For Expressions

```hcl
# Transform list
locals {
  names = ["alice", "bob", "charlie"]
  
  # List to list transformation
  upper_names = [for name in local.names : upper(name)]
  # Result: ["ALICE", "BOB", "CHARLIE"]
  
  # List with index
  indexed_names = [for idx, name in local.names : "${idx}: ${name}"]
  # Result: ["0: alice", "1: bob", "2: charlie"]
  
  # List to map
  name_map = { for name in local.names : name => upper(name) }
  # Result: { alice = "ALICE", bob = "BOB", charlie = "CHARLIE" }
  
  # Filtering (with if)
  long_names = [for name in local.names : name if length(name) > 4]
  # Result: ["alice", "charlie"]
}

# Transform map
locals {
  users = {
    alice   = { age = 25, role = "admin" }
    bob     = { age = 30, role = "user" }
    charlie = { age = 35, role = "admin" }
  }
  
  # Map to list
  user_names = [for name, info in local.users : name]
  # Result: ["alice", "bob", "charlie"]
  
  # Map to map (transformation)
  user_ages = { for name, info in local.users : name => info.age }
  # Result: { alice = 25, bob = 30, charlie = 35 }
  
  # Filtered map
  admins = { for name, info in local.users : name => info if info.role == "admin" }
  # Result: { alice = { age = 25, role = "admin" }, charlie = { age = 35, role = "admin" } }
}

# Grouping with for
locals {
  items = [
    { name = "a", group = "x" },
    { name = "b", group = "x" },
    { name = "c", group = "y" },
  ]
  
  # Group by (using ... to group)
  grouped = { for item in local.items : item.group => item.name... }
  # Result: { x = ["a", "b"], y = ["c"] }
}
```

### 3.9 Dynamic Blocks

Dynamic blocks generate repeated nested blocks:

```hcl
variable "ingress_rules" {
  type = list(object({
    port        = number
    protocol    = string
    cidr_blocks = list(string)
  }))
  
  default = [
    { port = 80, protocol = "tcp", cidr_blocks = ["0.0.0.0/0"] },
    { port = 443, protocol = "tcp", cidr_blocks = ["0.0.0.0/0"] },
    { port = 22, protocol = "tcp", cidr_blocks = ["10.0.0.0/8"] },
  ]
}

resource "azurerm_network_security_group" "example" {
  name                = "nsg-example"
  location            = var.location
  resource_group_name = var.resource_group_name

  # Dynamic security rules
  dynamic "security_rule" {
    for_each = var.ingress_rules
    
    content {
      name                       = "rule-${security_rule.key}"
      priority                   = 100 + security_rule.key
      direction                  = "Inbound"
      access                     = "Allow"
      protocol                   = security_rule.value.protocol
      source_port_range          = "*"
      destination_port_range     = security_rule.value.port
      source_address_prefixes    = security_rule.value.cidr_blocks
      destination_address_prefix = "*"
    }
  }
}

# Nested dynamic blocks
resource "azurerm_application_gateway" "example" {
  # ...
  
  dynamic "backend_http_settings" {
    for_each = var.backend_settings
    
    content {
      name                  = backend_http_settings.value.name
      cookie_based_affinity = "Disabled"
      port                  = backend_http_settings.value.port
      protocol              = "Http"
      
      dynamic "connection_draining" {
        for_each = backend_http_settings.value.drain_timeout != null ? [1] : []
        
        content {
          enabled           = true
          drain_timeout_sec = backend_http_settings.value.drain_timeout
        }
      }
    }
  }
}
```

---

## 4. Terraform Core Concepts

### 4.1 Resources

Resources are the most important element in Terraform - they describe infrastructure objects.

```hcl
# Resource syntax
resource "PROVIDER_TYPE" "LOCAL_NAME" {
  # Arguments specific to the resource type
  argument1 = "value1"
  argument2 = "value2"
}

# Example: Azure Resource Group
resource "azurerm_resource_group" "main" {
  name     = "rg-myapp-prod"
  location = "eastus"
  
  tags = {
    Environment = "production"
  }
}

# Referencing resource attributes
# Format: RESOURCE_TYPE.LOCAL_NAME.ATTRIBUTE
output "rg_id" {
  value = azurerm_resource_group.main.id
}
```

**Resource Meta-Arguments:**

```hcl
# 1. count - Create multiple instances
resource "azurerm_virtual_machine" "example" {
  count = 3
  
  name = "vm-${count.index}"  # count.index: 0, 1, 2
  # ...
}

# Reference: azurerm_virtual_machine.example[0], [1], [2]

# 2. for_each - Create instances from map or set
resource "azurerm_virtual_machine" "example" {
  for_each = {
    web  = "Standard_B1s"
    app  = "Standard_B2s"
    db   = "Standard_D4s_v3"
  }
  
  name = "vm-${each.key}"     # each.key: "web", "app", "db"
  size = each.value           # each.value: the sizes
  # ...
}

# Reference: azurerm_virtual_machine.example["web"], ["app"], ["db"]

# 3. depends_on - Explicit dependency
resource "azurerm_virtual_machine" "app" {
  # Explicit dependency (usually Terraform figures this out automatically)
  depends_on = [
    azurerm_postgresql_server.database,
    azurerm_key_vault_secret.db_password
  ]
  
  # ...
}

# 4. provider - Use specific provider configuration
provider "azurerm" {
  alias           = "west"
  subscription_id = "xxx-west-xxx"
  features {}
}

resource "azurerm_resource_group" "west" {
  provider = azurerm.west  # Use the "west" provider
  
  name     = "rg-west"
  location = "westus"
}

# 5. lifecycle - Customize resource behavior
resource "azurerm_virtual_machine" "example" {
  # ...
  
  lifecycle {
    # Don't destroy before creating replacement
    create_before_destroy = true
    
    # Prevent destruction (safety)
    prevent_destroy = true
    
    # Ignore changes to specific attributes
    ignore_changes = [
      tags,
      custom_data,
    ]
    
    # Replace when specific attribute changes
    replace_triggered_by = [
      azurerm_key_vault_secret.example.id
    ]
  }
}
```

### 4.2 Data Sources

Data sources fetch information about existing resources NOT managed by your Terraform configuration:

```hcl
# Fetch existing resource group
data "azurerm_resource_group" "existing" {
  name = "existing-rg"
}

# Fetch current subscription
data "azurerm_subscription" "current" {}

# Fetch existing virtual network
data "azurerm_virtual_network" "existing" {
  name                = "existing-vnet"
  resource_group_name = "networking-rg"
}

# Use data source in resource
resource "azurerm_subnet" "new" {
  name                 = "new-subnet"
  resource_group_name  = data.azurerm_resource_group.existing.name
  virtual_network_name = data.azurerm_virtual_network.existing.name
  address_prefixes     = ["10.0.1.0/24"]
}

# Fetch Azure AD user
data "azuread_user" "admin" {
  user_principal_name = "admin@contoso.com"
}

# Fetch Key Vault secret
data "azurerm_key_vault_secret" "example" {
  name         = "my-secret"
  key_vault_id = azurerm_key_vault.example.id
}

# Use secret value
resource "azurerm_postgresql_server" "example" {
  administrator_login_password = data.azurerm_key_vault_secret.example.value
  # ...
}
```

### 4.3 Providers

Providers are plugins that implement resource types:

```hcl
# Provider configuration in terraform block
terraform {
  required_version = ">= 1.0.0"
  
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
    azuread = {
      source  = "hashicorp/azuread"
      version = "~> 2.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.0"
    }
    helm = {
      source  = "hashicorp/helm"
      version = "~> 2.0"
    }
  }
}

# Provider configuration
provider "azurerm" {
  features {
    key_vault {
      purge_soft_delete_on_destroy    = true
      recover_soft_deleted_key_vaults = true
    }
    virtual_machine {
      delete_os_disk_on_deletion = true
    }
  }
  
  # Optional: specify subscription
  subscription_id = var.subscription_id
  tenant_id       = var.tenant_id
}

# Provider alias for multiple configurations
provider "azurerm" {
  alias           = "production"
  subscription_id = var.prod_subscription_id
  features {}
}

provider "azurerm" {
  alias           = "development"
  subscription_id = var.dev_subscription_id
  features {}
}

# Use aliased provider
resource "azurerm_resource_group" "prod" {
  provider = azurerm.production
  name     = "rg-prod"
  location = "eastus"
}

# Kubernetes provider configuration
provider "kubernetes" {
  host                   = azurerm_kubernetes_cluster.example.kube_config[0].host
  client_certificate     = base64decode(azurerm_kubernetes_cluster.example.kube_config[0].client_certificate)
  client_key             = base64decode(azurerm_kubernetes_cluster.example.kube_config[0].client_key)
  cluster_ca_certificate = base64decode(azurerm_kubernetes_cluster.example.kube_config[0].cluster_ca_certificate)
}

# Helm provider configuration
provider "helm" {
  kubernetes {
    host                   = azurerm_kubernetes_cluster.example.kube_config[0].host
    client_certificate     = base64decode(azurerm_kubernetes_cluster.example.kube_config[0].client_certificate)
    client_key             = base64decode(azurerm_kubernetes_cluster.example.kube_config[0].client_key)
    cluster_ca_certificate = base64decode(azurerm_kubernetes_cluster.example.kube_config[0].cluster_ca_certificate)
  }
}
```

**Version Constraints:**

```hcl
version = "3.0.0"    # Exact version
version = ">= 3.0"   # Minimum version
version = "~> 3.0"   # >= 3.0, < 4.0 (recommended)
version = ">= 3.0, < 4.0"  # Range
```

---

## 5. State Management

### 5.1 What is State?

Terraform state is how Terraform tracks the relationship between your configuration and the real-world resources.

```
State Purpose:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Configuration (.tf)    State File           Real World        │
│  ┌───────────────┐    ┌───────────────┐    ┌───────────────┐   │
│  │ resource "vm" │    │ "vm" → id123  │    │ VM id123      │   │
│  │   name = "x"  │    │   name = "x"  │    │   name = "x"  │   │
│  │   size = "B1" │    │   size = "B1" │    │   size = "B1" │   │
│  └───────────────┘    └───────────────┘    └───────────────┘   │
│                                                                 │
│  State provides:                                                │
│  ├── Mapping of config to real resource IDs                    │
│  ├── Cached attribute values                                   │
│  ├── Metadata (dependencies, provider config)                  │
│  └── Performance (no need to query every apply)                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 Local State

By default, state is stored locally in `terraform.tfstate`:

```json
{
  "version": 4,
  "terraform_version": "1.5.0",
  "serial": 10,
  "lineage": "abc123...",
  "outputs": {
    "resource_group_id": {
      "value": "/subscriptions/.../resourceGroups/my-rg",
      "type": "string"
    }
  },
  "resources": [
    {
      "mode": "managed",
      "type": "azurerm_resource_group",
      "name": "example",
      "provider": "provider[\"registry.terraform.io/hashicorp/azurerm\"]",
      "instances": [
        {
          "schema_version": 0,
          "attributes": {
            "id": "/subscriptions/.../resourceGroups/my-rg",
            "location": "eastus",
            "name": "my-rg",
            "tags": {}
          }
        }
      ]
    }
  ]
}
```

**Problems with Local State:**
1. Cannot collaborate with team members
2. State file may contain secrets
3. No state locking (concurrent modifications)
4. Risk of losing state file

### 5.3 Remote State (Backend)

Remote backends store state in a shared location:

```hcl
# Azure Storage Backend (recommended for Azure)
terraform {
  backend "azurerm" {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "tfstateaccount"
    container_name       = "tfstate"
    key                  = "myproject/terraform.tfstate"
    
    # Optional: Use SAS token or MSI
    # use_msi              = true
  }
}

# AWS S3 Backend
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "myproject/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"  # For state locking
  }
}

# Terraform Cloud Backend
terraform {
  cloud {
    organization = "my-org"
    
    workspaces {
      name = "my-workspace"
    }
  }
}
```

**Setting Up Azure Storage Backend:**

```bash
# Create storage account for state
az group create --name tfstate-rg --location eastus

az storage account create \
  --name tfstateaccount \
  --resource-group tfstate-rg \
  --location eastus \
  --sku Standard_LRS \
  --encryption-services blob

az storage container create \
  --name tfstate \
  --account-name tfstateaccount

# Initialize with backend
terraform init \
  -backend-config="storage_account_name=tfstateaccount" \
  -backend-config="container_name=tfstate" \
  -backend-config="key=myproject/terraform.tfstate" \
  -backend-config="resource_group_name=tfstate-rg"
```

### 5.4 State Locking

State locking prevents concurrent modifications:

```
State Locking Flow:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  User A: terraform apply                                        │
│  ├── Acquires lock on state                                     │
│  ├── Reads state                                                │
│  ├── Makes changes                                              │
│  ├── Writes state                                               │
│  └── Releases lock                                              │
│                                                                 │
│  User B: terraform apply (concurrent)                           │
│  ├── Tries to acquire lock                                      │
│  ├── BLOCKED (lock held by User A)                              │
│  └── Waits or fails                                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

Azure Storage provides native locking. For S3, use DynamoDB:

```hcl
# DynamoDB table for S3 state locking
resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}
```

### 5.5 State Commands

```bash
# List resources in state
terraform state list

# Show specific resource
terraform state show azurerm_resource_group.example

# Move resource to different address
terraform state mv azurerm_resource_group.old azurerm_resource_group.new

# Remove resource from state (resource still exists in cloud)
terraform state rm azurerm_resource_group.example

# Import existing resource into state
terraform import azurerm_resource_group.imported /subscriptions/.../resourceGroups/existing-rg

# Pull remote state to local
terraform state pull > state.json

# Push local state to remote (DANGEROUS)
terraform state push state.json

# Refresh state from real resources
terraform refresh

# Replace resource (force recreation)
terraform apply -replace=azurerm_virtual_machine.example

# Taint resource (deprecated, use -replace)
terraform taint azurerm_virtual_machine.example
terraform untaint azurerm_virtual_machine.example
```

### 5.6 State Isolation Strategies

```
Environment Isolation Strategies:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. Separate State Files (RECOMMENDED)                          │
│     ├── dev/terraform.tfstate                                   │
│     ├── staging/terraform.tfstate                               │
│     └── prod/terraform.tfstate                                  │
│                                                                 │
│  2. Workspaces (built-in)                                       │
│     └── Single config, multiple states                          │
│         ├── env:dev                                             │
│         ├── env:staging                                         │
│         └── env:prod                                            │
│                                                                 │
│  3. Directory Structure                                         │
│     ├── environments/                                           │
│     │   ├── dev/                                                │
│     │   │   ├── main.tf                                         │
│     │   │   └── terraform.tfvars                                │
│     │   ├── staging/                                            │
│     │   └── prod/                                               │
│     └── modules/                                                │
│         ├── networking/                                         │
│         ├── compute/                                            │
│         └── database/                                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. Modules

### 6.1 What are Modules?

Modules are reusable, self-contained packages of Terraform configurations:

```
Module Structure:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  modules/                                                       │
│  └── aks-cluster/                                               │
│      ├── main.tf          # Resources                           │
│      ├── variables.tf     # Input variables                     │
│      ├── outputs.tf       # Output values                       │
│      ├── providers.tf     # Provider requirements               │
│      ├── versions.tf      # Version constraints                 │
│      └── README.md        # Documentation                       │
│                                                                 │
│  Root Module:                                                   │
│  ├── main.tf                                                    │
│  │   └── module "aks" { source = "./modules/aks-cluster" }      │
│  ├── variables.tf                                               │
│  └── terraform.tfvars                                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 6.2 Creating a Module

**modules/aks-cluster/variables.tf:**
```hcl
variable "cluster_name" {
  description = "Name of the AKS cluster"
  type        = string
}

variable "location" {
  description = "Azure region"
  type        = string
}

variable "resource_group_name" {
  description = "Name of the resource group"
  type        = string
}

variable "kubernetes_version" {
  description = "Kubernetes version"
  type        = string
  default     = "1.28"
}

variable "node_pools" {
  description = "Node pool configurations"
  type = map(object({
    vm_size    = string
    node_count = number
    min_count  = optional(number)
    max_count  = optional(number)
    node_labels = optional(map(string), {})
    node_taints = optional(list(string), [])
  }))
  
  default = {
    system = {
      vm_size    = "Standard_D2s_v3"
      node_count = 1
    }
  }
}

variable "network_config" {
  description = "Network configuration"
  type = object({
    vnet_subnet_id  = string
    network_plugin  = optional(string, "azure")
    service_cidr    = optional(string, "10.0.0.0/16")
    dns_service_ip  = optional(string, "10.0.0.10")
  })
}

variable "tags" {
  description = "Tags to apply to resources"
  type        = map(string)
  default     = {}
}
```

**modules/aks-cluster/main.tf:**
```hcl
resource "azurerm_kubernetes_cluster" "this" {
  name                = var.cluster_name
  location            = var.location
  resource_group_name = var.resource_group_name
  dns_prefix          = var.cluster_name
  kubernetes_version  = var.kubernetes_version

  default_node_pool {
    name                = "system"
    vm_size             = var.node_pools["system"].vm_size
    node_count          = var.node_pools["system"].node_count
    min_count           = var.node_pools["system"].min_count
    max_count           = var.node_pools["system"].max_count
    enable_auto_scaling = var.node_pools["system"].min_count != null
    vnet_subnet_id      = var.network_config.vnet_subnet_id
    node_labels         = lookup(var.node_pools["system"], "node_labels", {})
    
    temporary_name_for_rotation = "systemtemp"
  }

  identity {
    type = "SystemAssigned"
  }

  network_profile {
    network_plugin    = var.network_config.network_plugin
    service_cidr      = var.network_config.service_cidr
    dns_service_ip    = var.network_config.dns_service_ip
    load_balancer_sku = "standard"
  }

  tags = var.tags
}

# Additional node pools
resource "azurerm_kubernetes_cluster_node_pool" "this" {
  for_each = { for k, v in var.node_pools : k => v if k != "system" }

  name                  = each.key
  kubernetes_cluster_id = azurerm_kubernetes_cluster.this.id
  vm_size               = each.value.vm_size
  node_count            = each.value.node_count
  min_count             = each.value.min_count
  max_count             = each.value.max_count
  enable_auto_scaling   = each.value.min_count != null
  vnet_subnet_id        = var.network_config.vnet_subnet_id
  node_labels           = each.value.node_labels
  node_taints           = each.value.node_taints

  tags = var.tags
}
```

**modules/aks-cluster/outputs.tf:**
```hcl
output "cluster_id" {
  description = "The ID of the AKS cluster"
  value       = azurerm_kubernetes_cluster.this.id
}

output "cluster_name" {
  description = "The name of the AKS cluster"
  value       = azurerm_kubernetes_cluster.this.name
}

output "kube_config" {
  description = "Kubernetes config for kubectl"
  value       = azurerm_kubernetes_cluster.this.kube_config_raw
  sensitive   = true
}

output "kube_config_host" {
  description = "Kubernetes API server host"
  value       = azurerm_kubernetes_cluster.this.kube_config[0].host
  sensitive   = true
}

output "kubelet_identity" {
  description = "Kubelet managed identity"
  value = {
    client_id   = azurerm_kubernetes_cluster.this.kubelet_identity[0].client_id
    object_id   = azurerm_kubernetes_cluster.this.kubelet_identity[0].object_id
    resource_id = azurerm_kubernetes_cluster.this.kubelet_identity[0].user_assigned_identity_id
  }
}

output "node_resource_group" {
  description = "The resource group containing cluster resources"
  value       = azurerm_kubernetes_cluster.this.node_resource_group
}
```

**modules/aks-cluster/versions.tf:**
```hcl
terraform {
  required_version = ">= 1.0.0"

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = ">= 3.0.0"
    }
  }
}
```

### 6.3 Using Modules

**main.tf (root module):**
```hcl
# Local module
module "aks" {
  source = "./modules/aks-cluster"

  cluster_name        = "myapp-aks-${var.environment}"
  location            = var.location
  resource_group_name = azurerm_resource_group.main.name
  kubernetes_version  = "1.28"

  node_pools = {
    system = {
      vm_size    = "Standard_D2s_v3"
      node_count = 1
      min_count  = 1
      max_count  = 3
    }
    worker = {
      vm_size    = "Standard_D4s_v3"
      node_count = 2
      min_count  = 2
      max_count  = 10
      node_labels = {
        "workload" = "app"
      }
    }
  }

  network_config = {
    vnet_subnet_id = azurerm_subnet.aks.id
  }

  tags = local.common_tags
}

# Use module outputs
output "aks_cluster_name" {
  value = module.aks.cluster_name
}

# Terraform Registry module
module "naming" {
  source  = "Azure/naming/azurerm"
  version = "0.4.0"
  
  prefix = ["myapp", var.environment]
}

# Git repository module
module "vpc" {
  source = "git::https://github.com/myorg/terraform-modules.git//vpc?ref=v1.2.0"
  
  cidr_block = "10.0.0.0/16"
}

# GitHub module (shorthand)
module "example" {
  source = "github.com/hashicorp/example?ref=v1.0.0"
}

# S3 bucket module
module "s3_module" {
  source = "s3::https://s3-eu-west-1.amazonaws.com/bucket/module.zip"
}
```

### 6.4 Module Best Practices

```
Module Design Principles:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. Single Responsibility                                       │
│     └── One module = one logical component                      │
│                                                                 │
│  2. Encapsulation                                               │
│     └── Hide implementation details, expose interfaces          │
│                                                                 │
│  3. Composability                                               │
│     └── Modules should work together                            │
│                                                                 │
│  4. Versioning                                                  │
│     └── Use semantic versioning for shared modules              │
│                                                                 │
│  5. Documentation                                               │
│     └── README with examples, inputs, outputs                   │
│                                                                 │
│  6. Sensible Defaults                                           │
│     └── Make common cases simple, complex cases possible        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

Directory Structure:
├── modules/
│   ├── networking/
│   │   ├── vnet/
│   │   ├── subnet/
│   │   └── nsg/
│   ├── compute/
│   │   ├── vm/
│   │   └── vmss/
│   ├── containers/
│   │   ├── aks/
│   │   └── acr/
│   └── database/
│       ├── postgresql/
│       └── redis/
├── environments/
│   ├── dev/
│   ├── staging/
│   └── prod/
└── examples/
    └── complete/
```

---

## 7. Workspaces

### 7.1 Understanding Workspaces

Workspaces allow managing multiple states with the same configuration:

```
Workspace Concept:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Same Configuration (.tf files)                                 │
│           │                                                     │
│           ├── Workspace: default                                │
│           │   └── terraform.tfstate                             │
│           │                                                     │
│           ├── Workspace: dev                                    │
│           │   └── terraform.tfstate.d/dev/terraform.tfstate     │
│           │                                                     │
│           ├── Workspace: staging                                │
│           │   └── terraform.tfstate.d/staging/terraform.tfstate │
│           │                                                     │
│           └── Workspace: prod                                   │
│               └── terraform.tfstate.d/prod/terraform.tfstate    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 7.2 Workspace Commands

```bash
# List workspaces
terraform workspace list

# Create new workspace
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

# Switch workspace
terraform workspace select dev

# Show current workspace
terraform workspace show

# Delete workspace (must switch away first)
terraform workspace select default
terraform workspace delete dev
```

### 7.3 Using Workspaces in Configuration

```hcl
# Reference current workspace
locals {
  environment = terraform.workspace
  
  # Environment-specific configurations
  config = {
    dev = {
      vm_size       = "Standard_B1s"
      node_count    = 1
      enable_backup = false
    }
    staging = {
      vm_size       = "Standard_B2s"
      node_count    = 2
      enable_backup = true
    }
    prod = {
      vm_size       = "Standard_D4s_v3"
      node_count    = 3
      enable_backup = true
    }
  }
  
  current_config = local.config[terraform.workspace]
}

resource "azurerm_resource_group" "main" {
  name     = "rg-myapp-${terraform.workspace}"
  location = var.location
  
  tags = {
    Environment = terraform.workspace
  }
}

resource "azurerm_kubernetes_cluster" "aks" {
  name = "aks-${terraform.workspace}"
  
  default_node_pool {
    vm_size    = local.current_config.vm_size
    node_count = local.current_config.node_count
  }
  
  # ...
}
```

### 7.4 Workspaces with Remote Backend

```hcl
# Azure backend with workspace prefix
terraform {
  backend "azurerm" {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "tfstateaccount"
    container_name       = "tfstate"
    key                  = "myproject.tfstate"
    # State files: myproject.tfstateenv:dev, myproject.tfstateenv:staging, etc.
  }
}

# S3 backend with workspace key prefix
terraform {
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "myproject/terraform.tfstate"
    region = "us-east-1"
    
    workspace_key_prefix = "environments"
    # State files: environments/dev/myproject/terraform.tfstate
  }
}
```

### 7.5 Workspaces vs Directory Structure

```
When to use WORKSPACES:
├── Same infrastructure, different instances
├── Quick environment switching
├── Simpler CI/CD
└── Limited configuration differences

When to use DIRECTORY STRUCTURE:
├── Different infrastructure per environment
├── Significant configuration differences
├── Different provider configurations
├── More explicit isolation
└── Team collaboration (different teams own different environments)

Hybrid Approach (Common):
├── environments/
│   ├── dev/
│   │   ├── main.tf (uses workspace for further isolation)
│   │   └── terraform.tfvars
│   ├── staging/
│   └── prod/
└── modules/
    └── (shared modules)
```

---

## 8. Providers

### 8.1 Provider Registry

The Terraform Registry (registry.terraform.io) hosts providers:

```
Popular Providers:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Cloud Providers:                                               │
│  ├── hashicorp/azurerm    - Azure Resource Manager              │
│  ├── hashicorp/aws        - Amazon Web Services                 │
│  ├── hashicorp/google     - Google Cloud Platform               │
│  └── hashicorp/azuread    - Azure Active Directory              │
│                                                                 │
│  Infrastructure:                                                │
│  ├── hashicorp/kubernetes - Kubernetes resources                │
│  ├── hashicorp/helm       - Helm charts                         │
│  ├── hashicorp/docker     - Docker containers                   │
│  └── hashicorp/vault      - HashiCorp Vault                     │
│                                                                 │
│  Utilities:                                                     │
│  ├── hashicorp/random     - Random values                       │
│  ├── hashicorp/null       - Null resources (triggers)           │
│  ├── hashicorp/local      - Local files                         │
│  ├── hashicorp/tls        - TLS certificates                    │
│  ├── hashicorp/http       - HTTP data sources                   │
│  └── hashicorp/time       - Time-based resources                │
│                                                                 │
│  Monitoring:                                                    │
│  ├── datadog/datadog      - Datadog                             │
│  ├── newrelic/newrelic    - New Relic                           │
│  └── grafana/grafana      - Grafana                             │
│                                                                 │
│  DevOps:                                                        │
│  ├── integrations/github  - GitHub                              │
│  ├── hashicorp/azuredevops - Azure DevOps                       │
│  └── jfrog/artifactory    - JFrog Artifactory                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 8.2 Provider Configuration Patterns

```hcl
# Multiple provider instances
provider "azurerm" {
  features {}
  alias           = "hub"
  subscription_id = var.hub_subscription_id
}

provider "azurerm" {
  features {}
  alias           = "spoke"
  subscription_id = var.spoke_subscription_id
}

# Pass provider to module
module "hub_network" {
  source = "./modules/network"
  providers = {
    azurerm = azurerm.hub
  }
}

module "spoke_network" {
  source = "./modules/network"
  providers = {
    azurerm = azurerm.spoke
  }
}

# Provider configuration block in module
# modules/network/providers.tf
terraform {
  required_providers {
    azurerm = {
      source                = "hashicorp/azurerm"
      version               = ">= 3.0"
      configuration_aliases = [azurerm.hub, azurerm.spoke]
    }
  }
}
```

### 8.3 Provider Authentication

```hcl
# Azure - Multiple authentication methods

# 1. Azure CLI (local development)
provider "azurerm" {
  features {}
  # Uses: az login
}

# 2. Service Principal with Client Secret
provider "azurerm" {
  features {}
  subscription_id = var.subscription_id
  tenant_id       = var.tenant_id
  client_id       = var.client_id
  client_secret   = var.client_secret
}

# 3. Service Principal with Certificate
provider "azurerm" {
  features {}
  subscription_id             = var.subscription_id
  tenant_id                   = var.tenant_id
  client_id                   = var.client_id
  client_certificate_path     = var.cert_path
  client_certificate_password = var.cert_password
}

# 4. Managed Identity (in Azure VMs, AKS, etc.)
provider "azurerm" {
  features {}
  use_msi = true
}

# 5. OIDC / Workload Identity (GitHub Actions, Azure DevOps)
provider "azurerm" {
  features {}
  use_oidc = true
}

# Environment variables (recommended for CI/CD)
# ARM_SUBSCRIPTION_ID
# ARM_TENANT_ID
# ARM_CLIENT_ID
# ARM_CLIENT_SECRET (or ARM_CLIENT_CERTIFICATE_PATH)
# ARM_USE_MSI=true
# ARM_USE_OIDC=true
```

---

## 9. Azure Provider Deep Dive

### 9.1 Common Azure Resources

```hcl
# Resource Group
resource "azurerm_resource_group" "example" {
  name     = "rg-myapp-prod"
  location = "eastus"
  tags     = local.common_tags
}

# Virtual Network
resource "azurerm_virtual_network" "example" {
  name                = "vnet-myapp"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  
  tags = local.common_tags
}

# Subnet
resource "azurerm_subnet" "example" {
  name                 = "snet-app"
  resource_group_name  = azurerm_resource_group.example.name
  virtual_network_name = azurerm_virtual_network.example.name
  address_prefixes     = ["10.0.1.0/24"]
  
  service_endpoints = [
    "Microsoft.Storage",
    "Microsoft.KeyVault",
    "Microsoft.Sql"
  ]
  
  delegation {
    name = "aks-delegation"
    service_delegation {
      name = "Microsoft.ContainerService/managedClusters"
    }
  }
}

# Network Security Group
resource "azurerm_network_security_group" "example" {
  name                = "nsg-app"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name

  security_rule {
    name                       = "AllowHTTPS"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "443"
    source_address_prefix      = "Internet"
    destination_address_prefix = "*"
  }
}

# NSG Association
resource "azurerm_subnet_network_security_group_association" "example" {
  subnet_id                 = azurerm_subnet.example.id
  network_security_group_id = azurerm_network_security_group.example.id
}
```

### 9.2 AKS with Terraform

```hcl
# Complete AKS deployment
resource "azurerm_kubernetes_cluster" "example" {
  name                = "aks-myapp-${var.environment}"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  dns_prefix          = "aks-myapp"
  kubernetes_version  = "1.28"
  
  sku_tier = var.environment == "prod" ? "Standard" : "Free"

  default_node_pool {
    name                = "system"
    vm_size             = "Standard_D2s_v3"
    node_count          = 1
    min_count           = 1
    max_count           = 3
    enable_auto_scaling = true
    vnet_subnet_id      = azurerm_subnet.aks.id
    
    only_critical_addons_enabled = true
    
    node_labels = {
      "nodepool" = "system"
    }
    
    upgrade_settings {
      max_surge = "33%"
    }
  }

  identity {
    type = "SystemAssigned"
  }

  network_profile {
    network_plugin    = "azure"
    network_policy    = "azure"
    service_cidr      = "10.1.0.0/16"
    dns_service_ip    = "10.1.0.10"
    load_balancer_sku = "standard"
    
    load_balancer_profile {
      managed_outbound_ip_count = 1
    }
  }

  azure_active_directory_role_based_access_control {
    managed                = true
    azure_rbac_enabled     = true
    admin_group_object_ids = [data.azuread_group.aks_admins.object_id]
  }

  oms_agent {
    log_analytics_workspace_id = azurerm_log_analytics_workspace.example.id
  }

  key_vault_secrets_provider {
    secret_rotation_enabled  = true
    secret_rotation_interval = "2m"
  }

  workload_identity_enabled = true
  oidc_issuer_enabled       = true

  maintenance_window {
    allowed {
      day   = "Sunday"
      hours = [0, 1, 2, 3, 4]
    }
  }

  auto_scaler_profile {
    scale_down_delay_after_add = "10m"
    scale_down_unneeded        = "10m"
  }

  tags = local.common_tags
}

# Worker node pool
resource "azurerm_kubernetes_cluster_node_pool" "worker" {
  name                  = "worker"
  kubernetes_cluster_id = azurerm_kubernetes_cluster.example.id
  vm_size               = "Standard_D4s_v3"
  node_count            = 2
  min_count             = 2
  max_count             = 10
  enable_auto_scaling   = true
  vnet_subnet_id        = azurerm_subnet.aks.id
  
  node_labels = {
    "nodepool" = "worker"
    "workload" = "app"
  }
  
  node_taints = []
  
  tags = local.common_tags
}

# Spot instance node pool
resource "azurerm_kubernetes_cluster_node_pool" "spot" {
  name                  = "spot"
  kubernetes_cluster_id = azurerm_kubernetes_cluster.example.id
  vm_size               = "Standard_D4s_v3"
  node_count            = 0
  min_count             = 0
  max_count             = 20
  enable_auto_scaling   = true
  vnet_subnet_id        = azurerm_subnet.aks.id
  
  priority        = "Spot"
  eviction_policy = "Delete"
  spot_max_price  = -1  # Pay up to on-demand price
  
  node_labels = {
    "kubernetes.azure.com/scalesetpriority" = "spot"
  }
  
  node_taints = [
    "kubernetes.azure.com/scalesetpriority=spot:NoSchedule"
  ]
  
  tags = local.common_tags
}
```

### 9.3 Azure Container Registry

```hcl
resource "azurerm_container_registry" "example" {
  name                = "acrmyapp${var.environment}"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  sku                 = "Premium"
  admin_enabled       = false

  georeplications {
    location                = "westeurope"
    zone_redundancy_enabled = true
  }

  network_rule_set {
    default_action = "Deny"
    
    ip_rule {
      action   = "Allow"
      ip_range = var.allowed_ip_range
    }
    
    virtual_network {
      action    = "Allow"
      subnet_id = azurerm_subnet.aks.id
    }
  }

  retention_policy {
    days    = 30
    enabled = true
  }

  trust_policy {
    enabled = true
  }

  tags = local.common_tags
}

# Allow AKS to pull from ACR
resource "azurerm_role_assignment" "aks_acr" {
  scope                = azurerm_container_registry.example.id
  role_definition_name = "AcrPull"
  principal_id         = azurerm_kubernetes_cluster.example.kubelet_identity[0].object_id
}
```

### 9.4 Key Vault

```hcl
data "azurerm_client_config" "current" {}

resource "azurerm_key_vault" "example" {
  name                        = "kv-myapp-${var.environment}"
  location                    = azurerm_resource_group.example.location
  resource_group_name         = azurerm_resource_group.example.name
  tenant_id                   = data.azurerm_client_config.current.tenant_id
  sku_name                    = "standard"
  soft_delete_retention_days  = 7
  purge_protection_enabled    = var.environment == "prod"
  
  enable_rbac_authorization = true

  network_acls {
    bypass                     = "AzureServices"
    default_action             = "Deny"
    ip_rules                   = var.allowed_ips
    virtual_network_subnet_ids = [azurerm_subnet.aks.id]
  }

  tags = local.common_tags
}

# Role assignment for secrets
resource "azurerm_role_assignment" "kv_secrets" {
  scope                = azurerm_key_vault.example.id
  role_definition_name = "Key Vault Secrets User"
  principal_id         = azurerm_user_assigned_identity.app.principal_id
}

# Store a secret
resource "azurerm_key_vault_secret" "example" {
  name         = "database-password"
  value        = random_password.db.result
  key_vault_id = azurerm_key_vault.example.id
  
  depends_on = [
    azurerm_role_assignment.kv_admin
  ]
}
```

### 9.5 Storage Account

```hcl
resource "azurerm_storage_account" "example" {
  name                     = "stmyapp${var.environment}"
  resource_group_name      = azurerm_resource_group.example.name
  location                 = azurerm_resource_group.example.location
  account_tier             = "Standard"
  account_replication_type = var.environment == "prod" ? "GRS" : "LRS"
  account_kind             = "StorageV2"
  
  min_tls_version                 = "TLS1_2"
  enable_https_traffic_only       = true
  allow_nested_items_to_be_public = false
  
  blob_properties {
    versioning_enabled = true
    
    delete_retention_policy {
      days = 30
    }
    
    container_delete_retention_policy {
      days = 7
    }
  }

  network_rules {
    default_action             = "Deny"
    ip_rules                   = var.allowed_ips
    virtual_network_subnet_ids = [azurerm_subnet.app.id]
    bypass                     = ["AzureServices", "Logging", "Metrics"]
  }

  tags = local.common_tags
}

resource "azurerm_storage_container" "example" {
  name                  = "data"
  storage_account_name  = azurerm_storage_account.example.name
  container_access_type = "private"
}
```

---

## 10. Advanced HCL Patterns

### 10.1 Conditionally Creating Resources

```hcl
# Using count for conditional creation
variable "create_bastion" {
  type    = bool
  default = false
}

resource "azurerm_bastion_host" "example" {
  count = var.create_bastion ? 1 : 0
  
  name                = "bastion-myapp"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  # ...
}

# Reference conditional resource
output "bastion_dns" {
  value = var.create_bastion ? azurerm_bastion_host.example[0].dns_name : null
}

# Using for_each with conditional
resource "azurerm_private_endpoint" "example" {
  for_each = var.enable_private_endpoints ? toset(["storage", "keyvault", "sql"]) : []
  
  name = "pe-${each.value}"
  # ...
}
```

### 10.2 Complex Variable Transformations

```hcl
variable "applications" {
  type = list(object({
    name        = string
    port        = number
    replicas    = number
    environment = map(string)
  }))
}

locals {
  # Flatten nested structure
  app_envs = flatten([
    for app in var.applications : [
      for key, value in app.environment : {
        app_name = app.name
        env_key  = key
        env_value = value
      }
    ]
  ])
  
  # Create map from list
  apps_by_name = { for app in var.applications : app.name => app }
  
  # Filter applications
  high_replica_apps = [
    for app in var.applications : app 
    if app.replicas >= 3
  ]
  
  # Transform to different structure
  app_ports = {
    for app in var.applications : 
    app.name => {
      container_port = app.port
      service_port   = 80
      target_port    = app.port
    }
  }
}
```

### 10.3 Using null_resource and Triggers

```hcl
# Execute command when specific resources change
resource "null_resource" "configure_app" {
  triggers = {
    # Re-run when these change
    cluster_id = azurerm_kubernetes_cluster.example.id
    app_config = sha256(jsonencode(var.app_config))
  }

  provisioner "local-exec" {
    command = <<-EOT
      az aks get-credentials --resource-group ${var.resource_group_name} --name ${azurerm_kubernetes_cluster.example.name}
      kubectl apply -f manifests/
    EOT
    
    interpreter = ["bash", "-c"]
  }

  depends_on = [azurerm_kubernetes_cluster_node_pool.worker]
}

# Time delay
resource "time_sleep" "wait_30_seconds" {
  depends_on = [azurerm_kubernetes_cluster.example]
  
  create_duration = "30s"
}

resource "null_resource" "after_delay" {
  depends_on = [time_sleep.wait_30_seconds]
  
  # ...
}
```

### 10.4 External Data Source

```hcl
# Call external program to get data
data "external" "git_commit" {
  program = ["bash", "-c", <<-EOT
    echo '{"commit": "'$(git rev-parse --short HEAD)'"}'
  EOT
  ]
}

locals {
  git_commit = data.external.git_commit.result.commit
}

# Use in resources
resource "azurerm_resource_group" "example" {
  name     = "rg-myapp"
  location = var.location
  
  tags = {
    GitCommit = local.git_commit
  }
}
```

### 10.5 Terraform Template Files

**templates/user_data.tpl:**
```bash
#!/bin/bash
set -e

# Configuration from Terraform
HOSTNAME="${hostname}"
ENVIRONMENT="${environment}"
APP_PORT="${app_port}"

%{ for key, value in environment_vars ~}
export ${key}="${value}"
%{ endfor ~}

# Setup application
echo "Setting up ${hostname} in ${environment}..."
```

**main.tf:**
```hcl
locals {
  user_data = templatefile("${path.module}/templates/user_data.tpl", {
    hostname    = "app-server"
    environment = var.environment
    app_port    = 8080
    environment_vars = {
      LOG_LEVEL = "info"
      API_URL   = "https://api.example.com"
    }
  })
}

resource "azurerm_virtual_machine" "example" {
  # ...
  custom_data = base64encode(local.user_data)
}
```

---

## 11. Testing Infrastructure Code

### 11.1 Validation

```hcl
# Variable validation
variable "environment" {
  type = string
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "cidr_block" {
  type = string
  
  validation {
    condition     = can(cidrhost(var.cidr_block, 0))
    error_message = "Must be a valid CIDR block."
  }
}

variable "email" {
  type = string
  
  validation {
    condition     = can(regex("^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$", var.email))
    error_message = "Must be a valid email address."
  }
}

# Preconditions and postconditions
resource "azurerm_kubernetes_cluster" "example" {
  # ...
  
  lifecycle {
    precondition {
      condition     = var.node_count >= 1
      error_message = "Node count must be at least 1."
    }
    
    postcondition {
      condition     = self.kube_config[0].host != ""
      error_message = "Cluster must have a valid API server endpoint."
    }
  }
}

# Check blocks (Terraform 1.5+)
check "api_health" {
  data "http" "api" {
    url = "https://${azurerm_kubernetes_cluster.example.fqdn}/healthz"
  }
  
  assert {
    condition     = data.http.api.status_code == 200
    error_message = "API health check failed"
  }
}
```

### 11.2 Terratest

Terratest is a Go library for testing Terraform:

```go
// test/aks_test.go
package test

import (
    "testing"
    "github.com/gruntwork-io/terratest/modules/terraform"
    "github.com/gruntwork-io/terratest/modules/k8s"
    "github.com/stretchr/testify/assert"
)

func TestAKSCluster(t *testing.T) {
    t.Parallel()

    terraformOptions := &terraform.Options{
        TerraformDir: "../examples/aks",
        Vars: map[string]interface{}{
            "environment": "test",
            "location":    "eastus",
        },
    }

    // Clean up at the end
    defer terraform.Destroy(t, terraformOptions)

    // Deploy
    terraform.InitAndApply(t, terraformOptions)

    // Get outputs
    clusterName := terraform.Output(t, terraformOptions, "cluster_name")
    resourceGroup := terraform.Output(t, terraformOptions, "resource_group_name")

    // Verify cluster exists
    assert.NotEmpty(t, clusterName)

    // Get kubeconfig
    kubeconfig := terraform.Output(t, terraformOptions, "kube_config")

    // Test Kubernetes connectivity
    options := k8s.NewKubectlOptions("", kubeconfig, "default")
    nodes := k8s.GetNodes(t, options)
    
    assert.GreaterOrEqual(t, len(nodes), 1)
}
```

**Running tests:**
```bash
cd test
go mod init test
go mod tidy
go test -v -timeout 30m
```

### 11.3 terraform-compliance

Policy testing with terraform-compliance:

```gherkin
# features/tags.feature
Feature: Resource tagging compliance

  Scenario: All resources must have required tags
    Given I have resource that supports tags defined
    Then it must contain tags
    And its value must contain Environment
    And its value must contain ManagedBy
    And its value must contain CostCenter

  Scenario: Environment tag must have valid value
    Given I have resource that supports tags defined
    When it has tags
    Then it must contain "Environment"
    And its value must match the "^(dev|staging|prod)$" regex

# features/security.feature
Feature: Security compliance

  Scenario: Storage accounts must use HTTPS
    Given I have azurerm_storage_account defined
    Then it must have enable_https_traffic_only
    And its value must be true

  Scenario: Key vaults must have purge protection in production
    Given I have azurerm_key_vault defined
    When its name contains "prod"
    Then it must have purge_protection_enabled
    And its value must be true
```

**Running terraform-compliance:**
```bash
# Generate plan
terraform plan -out=plan.out
terraform show -json plan.out > plan.json

# Run compliance tests
terraform-compliance -p plan.json -f features/
```

### 11.4 Checkov

Static analysis for security:

```bash
# Install
pip install checkov

# Scan Terraform files
checkov -d .

# Scan with specific framework
checkov -d . --framework terraform

# Output to JSON
checkov -d . -o json > results.json

# Skip specific checks
checkov -d . --skip-check CKV_AZURE_1,CKV_AZURE_2

# Custom policies
checkov -d . --external-checks-dir ./custom_policies
```

**Example Checkov output:**
```
Passed checks: 45, Failed checks: 3, Skipped checks: 0

Check: CKV_AZURE_35: "Ensure default network access rule for Storage Accounts is set to deny"
        FAILED for resource: azurerm_storage_account.example
        File: /main.tf:45-67

Check: CKV_AZURE_42: "Ensure Azure Key Vault has soft delete enabled"
        PASSED for resource: azurerm_key_vault.example
        File: /main.tf:70-90
```

---

## 12. CI/CD for Terraform

### 12.1 GitOps Workflow

```
Terraform GitOps Workflow:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Developer                                                      │
│     │                                                           │
│     ▼                                                           │
│  1. Create feature branch                                       │
│     │                                                           │
│     ▼                                                           │
│  2. Make changes to .tf files                                   │
│     │                                                           │
│     ▼                                                           │
│  3. Push to remote                                              │
│     │                                                           │
│     ▼                                                           │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ CI Pipeline (PR)                                         │   │
│  │ ├── terraform fmt -check                                 │   │
│  │ ├── terraform validate                                   │   │
│  │ ├── terraform plan                                       │   │
│  │ ├── Checkov/tfsec scan                                   │   │
│  │ ├── terraform-compliance                                 │   │
│  │ └── Post plan output to PR                               │   │
│  └──────────────────────────────────────────────────────────┘   │
│     │                                                           │
│     ▼                                                           │
│  4. Code review + Approve                                       │
│     │                                                           │
│     ▼                                                           │
│  5. Merge to main                                               │
│     │                                                           │
│     ▼                                                           │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ CD Pipeline (Main)                                       │   │
│  │ ├── terraform plan                                       │   │
│  │ ├── Manual approval (for prod)                           │   │
│  │ └── terraform apply -auto-approve                        │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 12.2 GitHub Actions

**.github/workflows/terraform.yml:**
```yaml
name: Terraform

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  TF_VERSION: '1.5.0'
  WORKING_DIR: 'environments/dev'

permissions:
  id-token: write   # For OIDC
  contents: read
  pull-requests: write

jobs:
  validate:
    name: Validate
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - name: Terraform Format Check
        run: terraform fmt -check -recursive

      - name: Terraform Init
        run: terraform init -backend=false
        working-directory: ${{ env.WORKING_DIR }}

      - name: Terraform Validate
        run: terraform validate
        working-directory: ${{ env.WORKING_DIR }}

  security-scan:
    name: Security Scan
    runs-on: ubuntu-latest
    needs: validate
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Run Checkov
        uses: bridgecrewio/checkov-action@master
        with:
          directory: ${{ env.WORKING_DIR }}
          framework: terraform
          output_format: sarif
          output_file_path: results.sarif

      - name: Upload SARIF
        uses: github/codeql-action/upload-sarif@v2
        if: always()
        with:
          sarif_file: results.sarif

  plan:
    name: Plan
    runs-on: ubuntu-latest
    needs: [validate, security-scan]
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - name: Azure Login (OIDC)
        uses: azure/login@v1
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Terraform Init
        run: terraform init
        working-directory: ${{ env.WORKING_DIR }}

      - name: Terraform Plan
        id: plan
        run: |
          terraform plan -no-color -out=tfplan 2>&1 | tee plan.txt
        working-directory: ${{ env.WORKING_DIR }}
        continue-on-error: true

      - name: Post Plan to PR
        uses: actions/github-script@v6
        if: github.event_name == 'pull_request'
        with:
          script: |
            const fs = require('fs');
            const plan = fs.readFileSync('${{ env.WORKING_DIR }}/plan.txt', 'utf8');
            const maxLength = 65000;
            const truncatedPlan = plan.length > maxLength 
              ? plan.substring(0, maxLength) + '\n... (truncated)'
              : plan;
            
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `## Terraform Plan\n\`\`\`\n${truncatedPlan}\n\`\`\``
            });

      - name: Upload Plan
        uses: actions/upload-artifact@v3
        with:
          name: tfplan
          path: ${{ env.WORKING_DIR }}/tfplan

  apply:
    name: Apply
    runs-on: ubuntu-latest
    needs: plan
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    environment: production  # Requires approval
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - name: Azure Login
        uses: azure/login@v1
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Download Plan
        uses: actions/download-artifact@v3
        with:
          name: tfplan
          path: ${{ env.WORKING_DIR }}

      - name: Terraform Init
        run: terraform init
        working-directory: ${{ env.WORKING_DIR }}

      - name: Terraform Apply
        run: terraform apply -auto-approve tfplan
        working-directory: ${{ env.WORKING_DIR }}
```

### 12.3 Azure DevOps Pipeline

**azure-pipelines.yml:**
```yaml
trigger:
  branches:
    include:
      - main
  paths:
    include:
      - 'terraform/**'

pr:
  branches:
    include:
      - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  - group: terraform-vars
  - name: TF_VERSION
    value: '1.5.0'
  - name: workingDirectory
    value: 'terraform/environments/$(environment)'

stages:
  - stage: Validate
    displayName: 'Validate'
    jobs:
      - job: Validate
        steps:
          - task: TerraformInstaller@1
            inputs:
              terraformVersion: $(TF_VERSION)

          - task: Bash@3
            displayName: 'Terraform Format'
            inputs:
              targetType: 'inline'
              script: 'terraform fmt -check -recursive'
              workingDirectory: 'terraform'

          - task: TerraformCLI@1
            displayName: 'Terraform Init'
            inputs:
              command: 'init'
              workingDirectory: $(workingDirectory)
              backendType: 'azurerm'
              backendServiceArm: 'Azure-Service-Connection'
              backendAzureRmResourceGroupName: 'tfstate-rg'
              backendAzureRmStorageAccountName: 'tfstateaccount'
              backendAzureRmContainerName: 'tfstate'
              backendAzureRmKey: '$(environment).tfstate'

          - task: TerraformCLI@1
            displayName: 'Terraform Validate'
            inputs:
              command: 'validate'
              workingDirectory: $(workingDirectory)

  - stage: Plan
    displayName: 'Plan'
    dependsOn: Validate
    jobs:
      - job: Plan
        steps:
          - task: TerraformInstaller@1
            inputs:
              terraformVersion: $(TF_VERSION)

          - task: TerraformCLI@1
            displayName: 'Terraform Init'
            inputs:
              command: 'init'
              workingDirectory: $(workingDirectory)
              backendType: 'azurerm'
              backendServiceArm: 'Azure-Service-Connection'
              backendAzureRmResourceGroupName: 'tfstate-rg'
              backendAzureRmStorageAccountName: 'tfstateaccount'
              backendAzureRmContainerName: 'tfstate'
              backendAzureRmKey: '$(environment).tfstate'

          - task: TerraformCLI@1
            displayName: 'Terraform Plan'
            inputs:
              command: 'plan'
              workingDirectory: $(workingDirectory)
              environmentServiceName: 'Azure-Service-Connection'
              publishPlanResults: 'terraform-plan'
              commandOptions: '-out=tfplan'

          - task: PublishPipelineArtifact@1
            inputs:
              targetPath: '$(workingDirectory)/tfplan'
              artifact: 'tfplan'

  - stage: Apply
    displayName: 'Apply'
    dependsOn: Plan
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - deployment: Apply
        environment: 'production'  # Requires approval
        strategy:
          runOnce:
            deploy:
              steps:
                - checkout: self

                - task: TerraformInstaller@1
                  inputs:
                    terraformVersion: $(TF_VERSION)

                - task: DownloadPipelineArtifact@2
                  inputs:
                    artifact: 'tfplan'
                    path: $(workingDirectory)

                - task: TerraformCLI@1
                  displayName: 'Terraform Init'
                  inputs:
                    command: 'init'
                    workingDirectory: $(workingDirectory)
                    backendType: 'azurerm'
                    backendServiceArm: 'Azure-Service-Connection'
                    backendAzureRmResourceGroupName: 'tfstate-rg'
                    backendAzureRmStorageAccountName: 'tfstateaccount'
                    backendAzureRmContainerName: 'tfstate'
                    backendAzureRmKey: '$(environment).tfstate'

                - task: TerraformCLI@1
                  displayName: 'Terraform Apply'
                  inputs:
                    command: 'apply'
                    workingDirectory: $(workingDirectory)
                    environmentServiceName: 'Azure-Service-Connection'
                    commandOptions: 'tfplan'
```

### 12.4 Jenkins Pipeline for Terraform

```groovy
pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: terraform
    image: hashicorp/terraform:1.5.0
    command: [sleep, infinity]
  - name: azure-cli
    image: mcr.microsoft.com/azure-cli
    command: [sleep, infinity]
'''
        }
    }
    
    environment {
        TF_IN_AUTOMATION = 'true'
        ARM_CLIENT_ID = credentials('azure-client-id')
        ARM_CLIENT_SECRET = credentials('azure-client-secret')
        ARM_SUBSCRIPTION_ID = credentials('azure-subscription-id')
        ARM_TENANT_ID = credentials('azure-tenant-id')
    }
    
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Target environment')
        booleanParam(name: 'AUTO_APPROVE', defaultValue: false, description: 'Auto approve apply')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Terraform Init') {
            steps {
                container('terraform') {
                    dir("environments/${params.ENVIRONMENT}") {
                        sh 'terraform init'
                    }
                }
            }
        }
        
        stage('Terraform Validate') {
            steps {
                container('terraform') {
                    dir("environments/${params.ENVIRONMENT}") {
                        sh 'terraform validate'
                    }
                }
            }
        }
        
        stage('Terraform Plan') {
            steps {
                container('terraform') {
                    dir("environments/${params.ENVIRONMENT}") {
                        sh 'terraform plan -out=tfplan'
                        sh 'terraform show -no-color tfplan > plan.txt'
                    }
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: "environments/${params.ENVIRONMENT}/plan.txt"
                }
            }
        }
        
        stage('Approval') {
            when {
                allOf {
                    branch 'main'
                    expression { !params.AUTO_APPROVE }
                }
            }
            steps {
                script {
                    def plan = readFile("environments/${params.ENVIRONMENT}/plan.txt")
                    input message: "Apply this plan?", 
                          parameters: [text(name: 'Plan', defaultValue: plan)]
                }
            }
        }
        
        stage('Terraform Apply') {
            when {
                branch 'main'
            }
            steps {
                container('terraform') {
                    dir("environments/${params.ENVIRONMENT}") {
                        sh 'terraform apply -auto-approve tfplan'
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        failure {
            emailext subject: "Terraform Pipeline Failed - ${params.ENVIRONMENT}",
                     body: "Pipeline failed. Check: ${BUILD_URL}",
                     recipientProviders: [requestor()]
        }
    }
}
```

---

## 13. Policy as Code

### 13.1 OPA (Open Policy Agent)

**policy/terraform.rego:**
```rego
package terraform.analysis

# Deny resources without required tags
deny[msg] {
    resource := input.resource_changes[_]
    resource.type == "azurerm_resource_group"
    not resource.change.after.tags.Environment
    msg := sprintf("Resource group '%s' must have Environment tag", [resource.address])
}

deny[msg] {
    resource := input.resource_changes[_]
    resource.type == "azurerm_resource_group"
    not resource.change.after.tags.CostCenter
    msg := sprintf("Resource group '%s' must have CostCenter tag", [resource.address])
}

# Deny storage accounts without HTTPS
deny[msg] {
    resource := input.resource_changes[_]
    resource.type == "azurerm_storage_account"
    not resource.change.after.enable_https_traffic_only
    msg := sprintf("Storage account '%s' must enforce HTTPS", [resource.address])
}

# Deny public AKS clusters in production
deny[msg] {
    resource := input.resource_changes[_]
    resource.type == "azurerm_kubernetes_cluster"
    contains(resource.address, "prod")
    resource.change.after.api_server_access_profile[_].authorized_ip_ranges == null
    msg := sprintf("Production AKS cluster '%s' must restrict API access", [resource.address])
}

# Warn about large VM sizes in non-prod
warn[msg] {
    resource := input.resource_changes[_]
    resource.type == "azurerm_virtual_machine"
    not contains(resource.address, "prod")
    contains(resource.change.after.vm_size, "Standard_D")
    msg := sprintf("Consider smaller VM size for non-prod: '%s'", [resource.address])
}
```

**Running OPA:**
```bash
# Generate plan JSON
terraform plan -out=tfplan
terraform show -json tfplan > tfplan.json

# Evaluate policy
opa eval --data policy/terraform.rego --input tfplan.json "data.terraform.analysis.deny"

# In CI/CD
result=$(opa eval --data policy/ --input tfplan.json --format raw "data.terraform.analysis.deny")
if [ "$result" != "[]" ]; then
    echo "Policy violations found:"
    echo "$result"
    exit 1
fi
```

### 13.2 Sentinel (HashiCorp)

Sentinel is HashiCorp's policy-as-code framework for Terraform Enterprise/Cloud:

```python
# policy/require-tags.sentinel
import "tfplan/v2" as tfplan

required_tags = ["Environment", "CostCenter", "ManagedBy"]

# Get all resources with tags
tagged_resources = filter tfplan.resource_changes as _, rc {
    rc.mode is "managed" and
    rc.type contains "azurerm" and
    (rc.change.actions contains "create" or rc.change.actions is ["update"])
}

# Check each resource has required tags
check_tags = rule {
    all tagged_resources as _, resource {
        all required_tags as tag {
            resource.change.after.tags contains tag
        }
    }
}

main = rule {
    check_tags
}
```

### 13.3 Azure Policy Integration

```hcl
# Define Azure Policy
resource "azurerm_policy_definition" "require_tag" {
  name         = "require-environment-tag"
  policy_type  = "Custom"
  mode         = "Indexed"
  display_name = "Require Environment tag"
  
  policy_rule = jsonencode({
    if = {
      allOf = [
        {
          field  = "tags['Environment']"
          exists = "false"
        }
      ]
    }
    then = {
      effect = "deny"
    }
  })
}

# Assign policy to subscription
resource "azurerm_subscription_policy_assignment" "require_tag" {
  name                 = "require-environment-tag"
  policy_definition_id = azurerm_policy_definition.require_tag.id
  subscription_id      = data.azurerm_subscription.current.id
  
  non_compliance_message {
    content = "Resources must have an Environment tag."
  }
}

# Built-in policy assignment
resource "azurerm_resource_group_policy_assignment" "allowed_locations" {
  name                 = "allowed-locations"
  resource_group_id    = azurerm_resource_group.example.id
  policy_definition_id = "/providers/Microsoft.Authorization/policyDefinitions/e56962a6-4747-49cd-b67b-bf8b01975c4c"
  
  parameters = jsonencode({
    listOfAllowedLocations = {
      value = ["eastus", "westus2", "westeurope"]
    }
  })
}
```

---

## 14. Best Practices

### 14.1 Project Structure

```
terraform-project/
├── .github/
│   └── workflows/
│       └── terraform.yml
├── modules/                    # Reusable modules
│   ├── networking/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── README.md
│   ├── aks/
│   ├── acr/
│   └── monitoring/
├── environments/               # Environment-specific configs
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   ├── backend.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   └── prod/
├── policies/                   # Policy as code
│   └── terraform.rego
├── tests/                      # Terratest
│   └── aks_test.go
├── .gitignore
├── .terraform-version
└── README.md
```

### 14.2 Naming Conventions

```hcl
# Resource naming
resource "azurerm_resource_group" "main" {
  name = "rg-${var.project}-${var.environment}-${var.location_short}"
  # Example: rg-myapp-prod-eus
}

resource "azurerm_kubernetes_cluster" "main" {
  name = "aks-${var.project}-${var.environment}"
  # Example: aks-myapp-prod
}

# Variable naming
variable "aks_node_count" {           # Specific, descriptive
  description = "Number of nodes in the default node pool"
  type        = number
  default     = 3
}

# Local naming
locals {
  common_tags = {                     # Grouped related values
    Project     = var.project
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
  
  is_production = var.environment == "prod"
}

# Output naming
output "aks_cluster_id" {              # resource_attribute format
  value = azurerm_kubernetes_cluster.main.id
}
```

### 14.3 Security Best Practices

```hcl
# 1. Never hardcode secrets
# BAD:
resource "azurerm_postgresql_server" "bad" {
  administrator_login_password = "SuperSecret123!"  # NEVER DO THIS
}

# GOOD:
resource "azurerm_postgresql_server" "good" {
  administrator_login_password = var.db_password  # From variable
}

# BETTER:
resource "azurerm_postgresql_server" "better" {
  administrator_login_password = data.azurerm_key_vault_secret.db_password.value
}

# 2. Mark sensitive variables
variable "db_password" {
  type      = string
  sensitive = true
}

# 3. Mark sensitive outputs
output "connection_string" {
  value     = azurerm_postgresql_server.example.connection_string
  sensitive = true
}

# 4. Use remote state with encryption
terraform {
  backend "azurerm" {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "tfstateaccount"
    container_name       = "tfstate"
    key                  = "terraform.tfstate"
    # Azure Storage encrypts by default
  }
}

# 5. Principle of least privilege
resource "azurerm_role_assignment" "example" {
  scope                = azurerm_key_vault.example.id
  role_definition_name = "Key Vault Secrets User"  # Not "Owner"
  principal_id         = azurerm_user_assigned_identity.example.principal_id
}
```

### 14.4 Performance Best Practices

```hcl
# 1. Use for_each instead of count when possible
# for_each is more stable for additions/removals
resource "azurerm_resource_group" "example" {
  for_each = toset(var.environments)
  name     = "rg-${each.key}"
  location = var.location
}

# 2. Limit provider interactions
# Use data sources sparingly - each is an API call
data "azurerm_subscription" "current" {}  # Call once

locals {
  subscription_id = data.azurerm_subscription.current.subscription_id
}

# 3. Use -target for debugging (not in production)
# terraform apply -target=module.aks

# 4. Parallelize with modules
module "app1" {
  source = "./modules/app"
  # ...
}

module "app2" {
  source = "./modules/app"
  # No dependency on app1 = parallel execution
}

# 5. Use refresh=false when you know state is current
# terraform plan -refresh=false
```

### 14.5 Documentation

```hcl
# Document all variables
variable "kubernetes_version" {
  description = <<-EOT
    The Kubernetes version for the AKS cluster.
    
    Supported versions can be found at:
    https://learn.microsoft.com/en-us/azure/aks/supported-kubernetes-versions
    
    Example: "1.28"
  EOT
  type    = string
  default = "1.28"
}

# Document complex locals
locals {
  # Calculate the CIDR blocks for subnets
  # We divide the VNet /16 into /24 subnets
  # First 10 subnets reserved for system use
  subnet_cidrs = {
    for idx, name in var.subnet_names :
    name => cidrsubnet(var.vnet_cidr, 8, idx + 10)
  }
}

# Document outputs
output "aks_oidc_issuer_url" {
  description = <<-EOT
    The OIDC issuer URL for the AKS cluster.
    
    Use this URL to configure Workload Identity federation
    with Azure AD or external identity providers.
  EOT
  value = azurerm_kubernetes_cluster.main.oidc_issuer_url
}
```

---

## 15. Troubleshooting

### 15.1 Common Errors

```
Error: Error acquiring the state lock
┌─────────────────────────────────────────────────────────────────┐
│ Cause: Another terraform process is running                    │
│                                                                 │
│ Solutions:                                                      │
│ 1. Wait for other process to complete                           │
│ 2. Force unlock (DANGEROUS):                                    │
│    terraform force-unlock <LOCK_ID>                             │
│ 3. Check who has the lock:                                      │
│    - Azure: Check blob lease in storage account                 │
│    - S3: Check DynamoDB table                                   │
└─────────────────────────────────────────────────────────────────┘

Error: Provider produced inconsistent result after apply
┌─────────────────────────────────────────────────────────────────┐
│ Cause: Resource changed outside of Terraform                   │
│                                                                 │
│ Solutions:                                                      │
│ 1. Run: terraform refresh                                       │
│ 2. Import the current state:                                    │
│    terraform import <address> <id>                              │
│ 3. Remove and recreate:                                         │
│    terraform state rm <address>                                 │
│    terraform apply                                              │
└─────────────────────────────────────────────────────────────────┘

Error: Resource already exists
┌─────────────────────────────────────────────────────────────────┐
│ Cause: Resource exists but not in Terraform state              │
│                                                                 │
│ Solutions:                                                      │
│ 1. Import existing resource:                                    │
│    terraform import azurerm_resource_group.example /sub/.../rg  │
│ 2. Delete existing resource manually                            │
│ 3. Use different name in configuration                          │
└─────────────────────────────────────────────────────────────────┘

Error: Cycle detected in dependency graph
┌─────────────────────────────────────────────────────────────────┐
│ Cause: Circular dependency between resources                   │
│                                                                 │
│ Solutions:                                                      │
│ 1. Review resource references                                   │
│ 2. Use depends_on to break cycles                               │
│ 3. Restructure resources                                        │
│ 4. Run: terraform graph | dot -Tpng > graph.png                 │
└─────────────────────────────────────────────────────────────────┘
```

### 15.2 Debugging

```bash
# Enable detailed logging
export TF_LOG=DEBUG
export TF_LOG_PATH=./terraform.log
terraform apply

# Log levels: TRACE, DEBUG, INFO, WARN, ERROR

# Provider-specific logging
export TF_LOG_PROVIDER=DEBUG

# Show dependency graph
terraform graph | dot -Tpng > graph.png

# Show state
terraform state show azurerm_resource_group.example

# List all resources
terraform state list

# Output in JSON
terraform output -json

# Validate syntax
terraform validate

# Check formatting
terraform fmt -check -diff
```

### 15.3 State Recovery

```bash
# Backup state before any risky operation
terraform state pull > state-backup.json

# Restore state (DANGEROUS)
terraform state push state-backup.json

# Move resource between states
# Source:
terraform state mv -state=source.tfstate azurerm_resource_group.old azurerm_resource_group.old
# Target:
terraform state mv -state=target.tfstate -state-out=target.tfstate azurerm_resource_group.old azurerm_resource_group.new

# Remove resource from state without destroying
terraform state rm azurerm_resource_group.example

# Import resource into state
terraform import azurerm_resource_group.example /subscriptions/.../resourceGroups/my-rg

# Replace corrupted state
# 1. List all resources manually
# 2. terraform state rm each one
# 3. terraform import each one
```

### 15.4 Common Patterns for Fixes

```hcl
# Ignore external changes
resource "azurerm_kubernetes_cluster" "example" {
  # ...
  
  lifecycle {
    ignore_changes = [
      default_node_pool[0].node_count,  # Managed by autoscaler
      tags["LastModified"],              # Set by automation
    ]
  }
}

# Force recreation
resource "azurerm_virtual_machine" "example" {
  # ...
  
  lifecycle {
    replace_triggered_by = [
      null_resource.force_replacement.id
    ]
  }
}

resource "null_resource" "force_replacement" {
  triggers = {
    # Change this value to force VM recreation
    force = "v2"
  }
}

# Prevent accidental deletion
resource "azurerm_resource_group" "production" {
  # ...
  
  lifecycle {
    prevent_destroy = true
  }
}

# Create before destroy
resource "azurerm_public_ip" "example" {
  # ...
  
  lifecycle {
    create_before_destroy = true
  }
}
```

---

## Summary

Terraform is a powerful Infrastructure as Code tool that enables:

1. **Declarative Infrastructure**: Define what you want, not how to create it
2. **Multi-Cloud Support**: Single tool for Azure, AWS, GCP, and more
3. **State Management**: Track real-world resources and detect drift
4. **Modules**: Reusable, composable infrastructure components
5. **CI/CD Integration**: Automated testing and deployment pipelines
6. **Policy as Code**: Enforce security and compliance automatically

Key best practices:
- Use remote state with locking
- Structure projects with modules and environments
- Implement CI/CD with plan reviews
- Test with Terratest and compliance tools
- Document thoroughly
- Never commit secrets

---

**Next Part**: [Part 9: System Design for Infrastructure](./DevOps-Complete-Reference-Guide-Part9-SystemDesign.md)
