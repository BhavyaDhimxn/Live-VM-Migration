# Terraform — Configuration

## ⚙️ Configuration Overview

Terraform configuration uses HCL (HashiCorp Configuration Language) to define infrastructure. Configuration files typically have `.tf` extension and can be organized in multiple files.

## 📁 Configuration Files

### 1. **Provider Configuration**
```hcl
# providers.tf
provider "aws" {
  region = var.aws_region
  
  # Optional: Use AWS profiles
  profile = "production"
  
  # Optional: Assume role
  assume_role {
    role_arn = "arn:aws:iam::123456789012:role/TerraformRole"
  }
  
  # Optional: Default tags
  default_tags {
    tags = {
      Environment = "production"
      ManagedBy   = "Terraform"
    }
  }
}

provider "azurerm" {
  features {
    resource_group {
      prevent_deletion_if_contains_resources = false
    }
  }
  
  subscription_id = var.azure_subscription_id
  client_id       = var.azure_client_id
  client_secret   = var.azure_client_secret
  tenant_id       = var.azure_tenant_id
}

provider "google" {
  project = var.gcp_project_id
  region  = var.gcp_region
  zone    = var.gcp_zone
}
```

### 2. **Backend Configuration**
```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "environments/production/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
    kms_key_id     = "arn:aws:kms:us-west-2:123456789012:key/abc123"
    
    # Optional: Workspace support
    workspace_key_prefix = "env:"
  }
}

# Alternative: Azure backend
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-state-rg"
    storage_account_name = "terraformstate"
    container_name       = "tfstate"
    key                  = "production.terraform.tfstate"
  }
}

# Alternative: GCS backend
terraform {
  backend "gcs" {
    bucket = "terraform-state-bucket"
    prefix = "production"
  }
}
```

### 3. **Variables Configuration**
```hcl
# variables.tf
variable "aws_region" {
  description = "AWS region for resources"
  type        = string
  default     = "us-west-2"
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
  
  validation {
    condition     = contains(["t2.micro", "t2.small", "t2.medium"], var.instance_type)
    error_message = "Instance type must be t2.micro, t2.small, or t2.medium."
  }
}

variable "environment" {
  description = "Environment name"
  type        = string
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "db_password" {
  description = "Database password"
  type        = string
  sensitive   = true
  
  validation {
    condition     = length(var.db_password) >= 12
    error_message = "Password must be at least 12 characters."
  }
}

variable "tags" {
  description = "Common tags for all resources"
  type        = map(string)
  default = {
    Project     = "MyProject"
    ManagedBy   = "Terraform"
    Environment = "production"
  }
}
```

### 4. **Outputs Configuration**
```hcl
# outputs.tf
output "instance_public_ip" {
  description = "Public IP address of the EC2 instance"
  value       = aws_instance.web.public_ip
  sensitive   = false
}

output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.web.id
}

output "vpc_id" {
  description = "ID of the VPC"
  value       = module.vpc.vpc_id
}

output "database_endpoint" {
  description = "RDS instance endpoint"
  value       = aws_db_instance.main.endpoint
  sensitive   = true
}
```

### 5. **Main Configuration**
```hcl
# main.tf
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-vpc"
    }
  )
}

resource "aws_subnet" "public" {
  count = length(var.public_subnet_cidrs)
  
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.public_subnet_cidrs[count.index]
  availability_zone = var.availability_zones[count.index]
  
  map_public_ip_on_launch = true
  
  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-public-subnet-${count.index + 1}"
    }
  )
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  subnet_id     = aws_subnet.public[0].id
  
  vpc_security_group_ids = [aws_security_group.web.id]
  
  user_data = <<-EOF
    #!/bin/bash
    apt-get update
    apt-get install -y nginx
    systemctl start nginx
    systemctl enable nginx
  EOF
  
  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-web-server"
    }
  )
}
```

## 🔧 Basic Configuration

### 1. **Data Sources**
```hcl
# data.tf
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
  
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

data "aws_availability_zones" "available" {
  state = "available"
}

data "aws_caller_identity" "current" {}

data "aws_region" "current" {}
```

### 2. **Local Values**
```hcl
# locals.tf
locals {
  common_tags = {
    Project     = "MyProject"
    ManagedBy   = "Terraform"
    Environment = var.environment
  }
  
  vpc_cidr = "10.0.0.0/16"
  
  public_subnet_cidrs = [
    cidrsubnet(local.vpc_cidr, 8, 0),
    cidrsubnet(local.vpc_cidr, 8, 1)
  ]
  
  private_subnet_cidrs = [
    cidrsubnet(local.vpc_cidr, 8, 2),
    cidrsubnet(local.vpc_cidr, 8, 3)
  ]
}
```

### 3. **Module Configuration**
```hcl
# modules/vpc/main.tf
variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
}

variable "environment" {
  description = "Environment name"
  type        = string
}

resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr
  
  tags = {
    Name        = "${var.environment}-vpc"
    Environment = var.environment
  }
}

output "vpc_id" {
  value = aws_vpc.main.id
}

# Usage in root module
module "vpc" {
  source = "./modules/vpc"
  
  vpc_cidr    = "10.0.0.0/16"
  environment = var.environment
}
```

## 🔐 Security Configuration

### 1. **Secrets Management**
```hcl
# Use environment variables
# export TF_VAR_db_password="secret-password"

# Or use .tfvars files (not in version control)
# terraform.tfvars
db_password = "secret-password"

# Or use AWS Secrets Manager
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "database/password"
}

resource "aws_db_instance" "main" {
  password = jsondecode(data.aws_secretsmanager_secret_version.db_password.secret_string)["password"]
}
```

### 2. **State Encryption**
```hcl
# Backend with encryption
terraform {
  backend "s3" {
    bucket         = "terraform-state"
    key            = "terraform.tfstate"
    encrypt        = true
    kms_key_id     = "arn:aws:kms:us-west-2:123456789012:key/abc123"
  }
}
```

### 3. **IAM Roles and Policies**
```hcl
# IAM role for Terraform
resource "aws_iam_role" "terraform" {
  name = "terraform-role"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "ec2.amazonaws.com"
      }
    }]
  })
}

resource "aws_iam_role_policy" "terraform" {
  role = aws_iam_role.terraform.id
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Action = [
        "ec2:*",
        "s3:*",
        "rds:*"
      ]
      Resource = "*"
    }]
  })
}
```

## 🔄 Advanced Configuration

### 1. **Workspaces**
```bash
# Create workspace
terraform workspace new production
terraform workspace new staging

# Switch workspace
terraform workspace select production

# List workspaces
terraform workspace list

# Use in configuration
resource "aws_instance" "web" {
  instance_type = terraform.workspace == "production" ? "t3.large" : "t2.micro"
}
```

### 2. **Conditional Resources**
```hcl
# Create resource conditionally
resource "aws_instance" "web" {
  count = var.create_instance ? 1 : 0
  
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
}
```

### 3. **For Each**
```hcl
# Create multiple resources
resource "aws_instance" "web" {
  for_each = var.instances
  
  ami           = data.aws_ami.ubuntu.id
  instance_type = each.value.instance_type
  
  tags = {
    Name = each.key
  }
}

# variables.tf
variable "instances" {
  type = map(object({
    instance_type = string
  }))
  default = {
    web1 = {
      instance_type = "t2.micro"
    }
    web2 = {
      instance_type = "t2.small"
    }
  }
}
```

## 🐛 Common Configuration Issues

### Issue 1: Provider Not Found
```bash
# Error: Could not find provider
# Solution: Initialize Terraform
terraform init
```

### Issue 2: State Lock
```bash
# Error: Error acquiring the state lock
# Solution: Force unlock (use with caution)
terraform force-unlock <lock-id>
```

### Issue 3: Backend Migration
```bash
# Migrate state to new backend
terraform init -migrate-state
```

## ✅ Configuration Validation

### Validate Configuration
```bash
# Validate syntax
terraform validate

# Format code
terraform fmt

# Check formatting
terraform fmt -check

# Validate with plan
terraform plan
```

### Configuration Checklist
- [ ] Provider configured
- [ ] Backend configured (for remote state)
- [ ] Variables defined
- [ ] Outputs defined
- [ ] Resources tagged appropriately
- [ ] Secrets managed securely
- [ ] State file secured

---

*Terraform configuration is now optimized! 🎯*