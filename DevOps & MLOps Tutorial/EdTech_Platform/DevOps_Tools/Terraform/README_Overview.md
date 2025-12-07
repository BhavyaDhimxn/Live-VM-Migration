# Terraform — Overview

## 🎯 What is Terraform?

**Terraform** is an open-source Infrastructure as Code (IaC) tool developed by HashiCorp that enables you to define and provision infrastructure using declarative configuration files. It supports multiple cloud providers and helps manage the complete infrastructure lifecycle through a consistent workflow.

## 🧩 Role in DevOps Lifecycle

Terraform plays a crucial role in the **Infrastructure Management** and **Provisioning** stages of the DevOps lifecycle:

- **🏗️ Infrastructure as Code**: Define infrastructure in version-controlled code
- **☁️ Multi-cloud Support**: Manage infrastructure across AWS, Azure, GCP, and more
- **🔄 State Management**: Track infrastructure state and changes
- **📦 Resource Provisioning**: Create, update, and destroy cloud resources
- **🔄 Change Management**: Plan and review infrastructure changes before applying
- **👥 Team Collaboration**: Share infrastructure definitions and collaborate
- **🔐 Security**: Manage infrastructure securely with secrets management
- **📊 Dependency Management**: Automatically handle resource dependencies

## 🚀 Key Components

### 1. **Configuration Files (.tf)**
```hcl
# main.tf
provider "aws" {
  region = "us-west-2"
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name = "WebServer"
    Environment = "production"
  }
}

resource "aws_security_group" "web" {
  name        = "web-sg"
  description = "Security group for web servers"
  
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

### 2. **State Management**
```bash
# Terraform maintains state to track resources
terraform state list
terraform state show aws_instance.web
terraform state mv aws_instance.web aws_instance.web_new
terraform state rm aws_instance.old
```

### 3. **Modules**
```hcl
# Use modules for reusability
module "vpc" {
  source = "./modules/vpc"
  
  cidr_block = "10.0.0.0/16"
  enable_dns = true
  tags = {
    Environment = "production"
  }
}

module "ec2" {
  source = "./modules/ec2"
  
  instance_type = "t2.micro"
  vpc_id        = module.vpc.vpc_id
  subnet_id     = module.vpc.public_subnet_id
}
```

### 4. **Variables and Outputs**
```hcl
# variables.tf
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "environment" {
  description = "Environment name"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

# outputs.tf
output "instance_ip" {
  description = "Public IP of the instance"
  value       = aws_instance.web.public_ip
}

output "instance_id" {
  description = "ID of the instance"
  value       = aws_instance.web.id
}
```

### 5. **Providers**
```hcl
# Multiple provider support
provider "aws" {
  region = "us-west-2"
}

provider "azurerm" {
  features {}
  subscription_id = var.azure_subscription_id
}

provider "google" {
  project = var.gcp_project_id
  region  = "us-central1"
}
```

## ⚙️ When to Use Terraform

### ✅ **Perfect For:**
- **Infrastructure as Code**: Version control infrastructure definitions
- **Multi-cloud Deployments**: Manage resources across multiple cloud providers
- **Reproducibility**: Recreate infrastructure consistently across environments
- **Team Collaboration**: Share and collaborate on infrastructure code
- **Change Management**: Plan and review changes before applying
- **Infrastructure Lifecycle**: Manage complete infrastructure lifecycle
- **Compliance**: Ensure infrastructure meets compliance requirements
- **Disaster Recovery**: Quickly recreate infrastructure after disasters

### ❌ **Not Ideal For:**
- **Simple Infrastructure**: Overhead for very simple, one-time setups
- **Dynamic Resources**: Resources that change very frequently
- **Application Code**: Not designed for application deployment
- **Real-time Configuration**: Not suitable for real-time configuration changes
- **Legacy Systems**: Limited support for legacy infrastructure

## 💡 Key Differentiators

| Feature | Terraform | Other IaC Tools |
|---------|-----------|-----------------|
| **Multi-cloud** | ✅ Native support | ⚠️ Limited |
| **State Management** | ✅ Built-in | ⚠️ External tools needed |
| **Declarative** | ✅ Pure declarative | ⚠️ Some imperative |
| **Provider Ecosystem** | ✅ 1000+ providers | ⚠️ Smaller ecosystem |
| **Plan Before Apply** | ✅ Always shows plan | ⚠️ Varies |
| **Open Source** | ✅ Free & open | ⚠️ Some commercial |
| **Community** | ✅ Large community | ⚠️ Smaller |

## 🔗 Integration Ecosystem

### Cloud Providers
- **AWS**: Complete AWS resource support
- **Azure**: Full Azure resource management
- **Google Cloud**: GCP resource provisioning
- **DigitalOcean**: DO resource management
- **Oracle Cloud**: OCI support
- **Alibaba Cloud**: Alibaba Cloud support

### Infrastructure Tools
- **Docker**: Container infrastructure
- **Kubernetes**: K8s resource management
- **VMware**: vSphere integration
- **OpenStack**: OpenStack support

### CI/CD Integration
- **Jenkins**: Terraform plugin
- **GitLab CI**: Terraform integration
- **GitHub Actions**: Terraform actions
- **CircleCI**: Terraform orb
- **Azure DevOps**: Terraform tasks

## 📈 Benefits for DevOps Teams

### 1. **🔄 Version Control**
```bash
# Infrastructure as code in Git
git init
git add *.tf
git commit -m "Add infrastructure configuration"
```

### 2. **📊 Change Visibility**
```bash
# See what will change before applying
terraform plan

# Output shows:
# + resource to be created
# - resource to be destroyed
# ~ resource to be modified
```

### 3. **🚀 Automation**
```bash
# Automate infrastructure provisioning
terraform init
terraform plan -out=tfplan
terraform apply tfplan
```

### 4. **🔐 Security**
```hcl
# Use variables and secrets management
variable "db_password" {
  type        = string
  sensitive   = true
  description = "Database password"
}

# Use backend for state encryption
terraform {
  backend "s3" {
    bucket         = "terraform-state"
    key            = "project/terraform.tfstate"
    encrypt        = true
    kms_key_id     = "arn:aws:kms:..."
  }
}
```

---

*Terraform provides powerful Infrastructure as Code capabilities! 🎯*