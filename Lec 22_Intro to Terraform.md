# Lec 22: Intro to Terraform

---

## What is Terraform?

**Terraform** = Infrastructure as Code (IaC) tool by HashiCorp.

**Purpose:** Define and manage infrastructure using **code** instead of manual clicking.

**Key features:**
- **Declarative**: Describe *what* you want, not *how* to create it
- **Multi-cloud**: AWS, Azure, GCP, Docker, Kubernetes
- **State management**: Tracks infrastructure in state file
- **Version controlled**: Infrastructure code in Git

---

## Infrastructure as Code (IaC)

**Traditional approach problems:**
- Manual provisioning → slow
- Human errors
- Can't reproduce environments easily
- No version control

**IaC benefits:**
- Infrastructure in Git → version controlled
- Consistent deployments
- Fast provisioning
- Reduced errors
- Easy collaboration

---

## Terraform Core Concepts

### 1. Providers
Plugins to interact with cloud APIs.

Examples: AWS, Azure, Docker, Kubernetes

### 2. Resources
Infrastructure components you create.

Examples: EC2 instance, S3 bucket, Docker container

### 3. State
File that maps Terraform config to real infrastructure.

Tracks what exists, what needs to change.

### 4. Execution Plan
Preview of changes before applying them.

---

## Terraform Workflow

**4 core commands:**

```bash
# 1. Initialize - download providers
terraform init

# 2. Plan - preview changes
terraform plan

# 3. Apply - create/update infrastructure
terraform apply

# 4. Destroy - delete all resources
terraform destroy
```

**Flow:**
```
Write .tf file → terraform init → terraform plan → terraform apply
```

---

## HCL Syntax (HashiCorp Configuration Language)

**Files:** `.tf` extension

**Structure:**
```hcl
block_type "label1" "label2" {
  argument1 = value1
  argument2 = value2
}
```

**Resource syntax:**
```hcl
resource "<provider>_<resource_type>" "<resource_name>" {
  argument = value
}
```

---

## Simple Example: Local File

**main.tf:**
```hcl
resource "local_file" "example" {
  filename = "/tmp/terraform.txt"
  content  = "Hello from Terraform!"
}
```

**Execute:**
```bash
terraform init
terraform plan
terraform apply
```

**Result:** File created at `/tmp/terraform.txt`

---

## Docker Example

**main.tf:**
```hcl
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 2.21.0"
    }
  }
}

provider "docker" {}

resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = false
}

resource "docker_container" "nginx_container" {
  image = docker_image.nginx.latest
  name  = "terraform-nginx"

  ports {
    internal = 80
    external = 8080
  }
}
```

**Execute:**
```bash
terraform init      # Downloads Docker provider
terraform apply     # Creates nginx container
```

---

## Variables

**variables.tf:**
```hcl
variable "filename" {
  type    = string
  default = "/tmp/myfile.txt"
}

variable "content" {
  type    = string
  default = "Content from variable"
}
```

**main.tf:**
```hcl
resource "local_file" "file" {
  filename = var.filename
  content  = var.content
}
```

**Data types:**
- `string`
- `number`
- `bool`
- `list(string)`
- `map(string)`

---

## Outputs

Display values after apply.

**outputs.tf:**
```hcl
output "container_name" {
  value = docker_container.nginx_container.name
}

output "container_ip" {
  value = docker_container.nginx_container.ip_address
}
```

**Use cases:**
- Debugging
- Share values between modules
- Display IPs, endpoints

---

## Useful Commands

```bash
terraform fmt          # Format .tf files
terraform validate     # Check syntax
terraform show         # Display state/plan
terraform state list   # List managed resources
terraform destroy      # Delete all resources
```

---

## Terraform vs Other Tools

| Tool | Purpose |
|------|---------|
| **Terraform** | Infrastructure provisioning |
| **Ansible** | Configuration management (install software) |
| **Docker** | Containerization |
| **Vagrant** | VM management |

**Common pattern:**
1. **Terraform** creates infrastructure (EC2, VPC)
2. **Ansible** configures software on servers
3. **Docker** runs applications in containers

---

## Important Notes

**terraform init:**
- Downloads providers
- Initializes backend
- Locks provider versions
- Run once per directory

**terraform plan:**
- Shows what will change
- **Always run before apply**
- Prevents accidental changes

**terraform apply:**
- Executes changes
- Updates state file
- Requires confirmation (or use `-auto-approve`)

**State file:**
- `terraform.tfstate`
- Tracks current infrastructure
- **Never edit manually**
- Sensitive data inside (passwords, keys)

**Best practices:**
- Use version control (Git) for `.tf` files
- Don't commit `terraform.tfstate` (contains secrets)
- Use remote state (S3, Terraform Cloud) for teams
- Always run `plan` before `apply`
