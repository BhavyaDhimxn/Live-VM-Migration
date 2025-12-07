# Terraform — Best Practices

## 🎯 Production Best Practices

### 1. **File Organization**
```
terraform-project/
├── environments/
│   ├── production/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   └── staging/
│       ├── main.tf
│       ├── variables.tf
│       └── terraform.tfvars
├── modules/
│   ├── vpc/
│   ├── ec2/
│   └── rds/
├── .gitignore
└── README.md
```

### 2. **Naming Conventions**
```hcl
# Use consistent naming
# Format: {resource_type}_{purpose}_{environment}
# Examples:
resource "aws_instance" "web_production" {}
resource "aws_security_group" "app_staging" {}
resource "aws_s3_bucket" "logs_production" {}
```

## 🔐 Security Best Practices

### 1. **State Management**
```hcl
# Use remote backend with encryption
terraform {
  backend "s3" {
    bucket         = "terraform-state-bucket"
    key            = "project/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
    kms_key_id     = "arn:aws:kms:us-west-2:123456789012:key/abc123"
  }
}
```

### 2. **Secrets Management**
```hcl
# Never hardcode secrets
# Bad:
resource "aws_db_instance" "main" {
  password = "hardcoded-password"
}

# Good: Use variables
variable "db_password" {
  type        = string
  sensitive   = true
  description = "Database password"
}

# Or use AWS Secrets Manager
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "database/password"
}

resource "aws_db_instance" "main" {
  password = jsondecode(data.aws_secretsmanager_secret_version.db_password.secret_string)["password"]
}
```

### 3. **IAM Best Practices**
```hcl
# Use least privilege IAM policies
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
        "ec2:DescribeInstances",
        "ec2:RunInstances",
        "ec2:TerminateInstances"
      ]
      Resource = "*"
    }]
  })
}
```

## 📊 Code Organization Best Practices

### 1. **Use Modules**
```hcl
# Create reusable modules
module "vpc" {
  source = "./modules/vpc"
  
  vpc_cidr    = "10.0.0.0/16"
  environment = var.environment
}

# Use public modules from Terraform Registry
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  
  name = "my-vpc"
  cidr = "10.0.0.0/16"
}
```

### 2. **Variable Validation**
```hcl
# Validate variables
variable "instance_type" {
  type        = string
  description = "EC2 instance type"
  
  validation {
    condition     = contains(["t2.micro", "t2.small", "t3.micro"], var.instance_type)
    error_message = "Instance type must be t2.micro, t2.small, or t3.micro."
  }
}

variable "environment" {
  type        = string
  description = "Environment name"
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}
```

### 3. **Output Documentation**
```hcl
# Document all outputs
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.web.id
}

output "instance_public_ip" {
  description = "Public IP address of the EC2 instance"
  value       = aws_instance.web.public_ip
  sensitive   = false
}
```

## 🔄 CI/CD Integration

### 1. **Automated Terraform Execution**
```yaml
# .github/workflows/terraform.yml
name: Terraform

on:
  push:
    branches: [main]
    paths:
      - 'terraform/**'

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v1
      with:
        terraform_version: 1.6.0
    
    - name: Terraform Init
      run: terraform init
      working-directory: ./terraform
    
    - name: Terraform Plan
      run: terraform plan
      working-directory: ./terraform
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    
    - name: Terraform Apply
      if: github.ref == 'refs/heads/main'
      run: terraform apply -auto-approve
      working-directory: ./terraform
```

## 📈 Performance Optimization

### 1. **Parallelism**
```bash
# Control parallelism
terraform apply -parallelism=10

# Default is 10, adjust based on provider limits
```

### 2. **Targeted Operations**
```bash
# Apply specific resources only
terraform apply -target=aws_instance.web

# Plan specific resources
terraform plan -target=aws_instance.web
```

### 3. **State Optimization**
```bash
# Use state locking
terraform {
  backend "s3" {
    dynamodb_table = "terraform-state-lock"
  }
}

# Compact state
terraform state pull > terraform.tfstate
terraform state push terraform.tfstate
```

## 🔧 Reproducibility Best Practices

### 1. **Version Pinning**
```hcl
# Pin provider versions
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

### 2. **Module Versioning**
```hcl
# Use versioned modules
module "vpc" {
  source = "git::https://github.com/example/terraform-modules.git//vpc?ref=v1.0.0"
}
```

### 3. **State Versioning**
```hcl
# Enable state versioning in S3
resource "aws_s3_bucket_versioning" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  
  versioning_configuration {
    status = "Enabled"
  }
}
```

## 🧪 Testing Best Practices

### 1. **Validation**
```bash
# Validate configuration
terraform validate

# Format code
terraform fmt -check

# Check in CI/CD
terraform fmt -check -diff
```

### 2. **Plan Review**
```bash
# Always review plan before apply
terraform plan -out=tfplan

# Review plan file
terraform show tfplan

# Apply only after review
terraform apply tfplan
```

### 3. **State Validation**
```bash
# Verify state consistency
terraform state list

# Check for drift
terraform plan -detailed-exitcode
```

## 📚 Documentation Best Practices

### 1. **Code Comments**
```hcl
# Create VPC for production environment
# This VPC uses 10.0.0.0/16 CIDR block
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  
  # Enable DNS resolution
  enable_dns_hostnames = true
  enable_dns_support   = true
}
```

### 2. **README Documentation**
```markdown
# Terraform Infrastructure

## Overview
This directory contains Terraform configurations for...

## Prerequisites
- Terraform >= 1.0
- AWS CLI configured
- Appropriate IAM permissions

## Usage
```bash
terraform init
terraform plan
terraform apply
```

## Variables
- `environment`: Environment name (dev/staging/prod)
- `instance_type`: EC2 instance type
```

## 🚨 Common Pitfalls to Avoid

### 1. **Don't Commit Secrets**
```bash
# Bad: Committing secrets
# terraform.tfvars
db_password = "secret123"

# Good: Use .gitignore
echo "*.tfvars" >> .gitignore
echo ".terraform/" >> .gitignore
echo "terraform.tfstate*" >> .gitignore
```

### 2. **Don't Use Local State in Production**
```hcl
# Bad: Local state
# terraform {
#   backend "local" {}
# }

# Good: Remote state
terraform {
  backend "s3" {
    bucket = "terraform-state"
    key    = "terraform.tfstate"
  }
}
```

### 3. **Don't Hardcode Values**
```hcl
# Bad: Hardcoded values
resource "aws_instance" "web" {
  instance_type = "t2.micro"
  ami           = "ami-0c55b159cbfafe1f0"
}

# Good: Use variables
variable "instance_type" {
  type    = string
  default = "t2.micro"
}

resource "aws_instance" "web" {
  instance_type = var.instance_type
  ami           = data.aws_ami.ubuntu.id
}
```

---

*Follow these best practices to build robust infrastructure with Terraform! 🎯*