# Lec 23: Terraform Continued

---

## Terraform State

**State file:** `terraform.tfstate` - maps config to real infrastructure.

**Contains:** Resource IDs, attributes, dependencies.

**Critical:** Terraform cannot work without state.

**State locking:** Prevents concurrent modifications. Manual unlock: `terraform force-unlock LOCK_ID`

**Warning:** State file contains plaintext secrets (DB passwords, API keys).

---

## Backends: Local vs Remote

### Local Backend
```hcl
terraform {
  backend "local" {
    path = "terraform.tfstate"
  }
}
```
Not for teams.

### Remote Backend (S3 + DynamoDB)
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

**Benefits:** Team collaboration, state locking, centralized storage, encryption.

---

## AWS Provider Setup

**Export credentials:**
```bash
export AWS_ACCESS_KEY_ID=<your_key>
export AWS_SECRET_ACCESS_KEY=<your_secret>
```

**Provider:**
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}
```

---

## AWS VPC and Subnets

**VPC:**
```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}
```

**Subnet:**
```hcl
resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
}
```

**Implicit dependency:** Terraform knows subnet depends on VPC.

**IP types:**
- Private IP: Internal only
- Public IP: Temporary
- Elastic IP: Permanent

---

## EC2 Instance

```hcl
resource "aws_instance" "server" {
  ami           = "ami-08c40ec9ead489470"
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.public.id

  tags = {
    Name = "TerraformServer"
  }
}

output "instance_ip" {
  value = aws_instance.server.public_ip
}
```

---

## State Commands

```bash
terraform state list                    # List resources
terraform state show aws_instance.server # Show details
terraform state rm aws_instance.server   # Remove from state
```

---

## Meta-Arguments

### count
```hcl
resource "aws_instance" "server" {
  count         = 3
  ami           = "ami-xxxx"
  instance_type = "t2.micro"
  
  tags = {
    Name = "Server-${count.index}"
  }
}
```
Access: `aws_instance.server[0]`

### for_each
```hcl
variable "instances" {
  default = {
    web = "t2.micro"
    db  = "t2.small"
  }
}

resource "aws_instance" "server" {
  for_each      = var.instances
  ami           = "ami-xxxx"
  instance_type = each.value
  
  tags = {
    Name = each.key
  }
}
```
Access: `aws_instance.server["web"]`

### depends_on
```hcl
resource "aws_instance" "server" {
  ami           = "ami-xxxx"
  instance_type = "t2.micro"
  depends_on    = [aws_s3_bucket.logs]
}
```

---

## Terraform Modules

**Structure:**
```
modules/ec2/
├── main.tf
├── variables.tf
└── outputs.tf
```

**Usage:**
```hcl
module "web_server" {
  source        = "./modules/ec2"
  ami           = "ami-xxxx"
  instance_type = "t2.micro"
}

output "web_ip" {
  value = module.web_server.public_ip
}
```

**Benefits:** Code reuse, organization.

---

## Provisioners

**Use only as last resort.** Prefer user-data.

### local-exec
```hcl
provisioner "local-exec" {
  command = "echo ${self.public_ip} > ip.txt"
}
```

### remote-exec
```hcl
provisioner "remote-exec" {
  inline = [
    "sudo apt update",
    "sudo apt install -y nginx"
  ]
  
  connection {
    type        = "ssh"
    user        = "ubuntu"
    private_key = file("~/.ssh/id_rsa")
    host        = self.public_ip
  }
}
```

**Failure:** Resource marked as tainted, recreated on next apply.

---

## Useful Functions

```hcl
# String
upper("hello")                    # "HELLO"

# Numeric
max(5, 10, 15)                    # 15

# Collection
length([1, 2, 3])                 # 3
element(["a", "b"], 1)            # "b"

# File
file("path/to/file")              # Read file

# Network
cidrsubnet("10.0.0.0/16", 8, 1)   # "10.0.1.0/24"
```

---

## Debugging

```bash
export TF_LOG=TRACE
export TF_LOG_PATH=terraform.log
terraform apply
```

**Levels:** TRACE, DEBUG, INFO, WARN, ERROR

---

## Important Notes

**State:**
- Never edit manually
- Use remote backend for teams
- Contains secrets - secure it
- S3 + DynamoDB for locking

**Meta-arguments:**
- `count` for repetition
- `for_each` for named resources
- `depends_on` for explicit ordering

**Provisioners:**
- Last resort only
- Use user-data instead
- Remote-exec needs SSH connection

