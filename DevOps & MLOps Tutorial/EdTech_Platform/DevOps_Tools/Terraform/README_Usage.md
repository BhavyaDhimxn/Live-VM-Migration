# Terraform — Usage

## 🚀 Getting Started with Terraform

This guide covers practical usage examples for Terraform in real-world infrastructure provisioning scenarios.

## 📊 Example 1: Create AWS EC2 Instance

### Scenario: Provision a Web Server

Create a simple EC2 instance with security group for a web server.

### Step 1: Create Configuration Files
```hcl
# main.tf
provider "aws" {
  region = "us-west-2"
}

data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}

resource "aws_security_group" "web" {
  name        = "web-sg"
  description = "Security group for web server"
  
  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  ingress {
    description = "HTTPS"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "web-sg"
  }
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
  
  vpc_security_group_ids = [aws_security_group.web.id]
  
  user_data = <<-EOF
    #!/bin/bash
    apt-get update
    apt-get install -y nginx
    systemctl start nginx
    systemctl enable nginx
  EOF
  
  tags = {
    Name = "WebServer"
    Environment = "production"
  }
}

# outputs.tf
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.web.id
}

output "instance_public_ip" {
  description = "Public IP address of the EC2 instance"
  value       = aws_instance.web.public_ip
}

output "instance_public_dns" {
  description = "Public DNS name of the EC2 instance"
  value       = aws_instance.web.public_dns
}
```

### Step 2: Initialize Terraform
```bash
# Initialize Terraform (downloads AWS provider)
terraform init

# Expected output:
# Initializing provider plugins...
# - Finding latest version of hashicorp/aws...
# - Installing hashicorp/aws v5.0.0...
# Terraform has been successfully initialized!
```

### Step 3: Plan Changes
```bash
# Review what will be created
terraform plan

# Save plan to file
terraform plan -out=tfplan

# Expected output shows:
# + aws_instance.web
# + aws_security_group.web
```

### Step 4: Apply Configuration
```bash
# Apply the configuration
terraform apply

# Or apply saved plan
terraform apply tfplan

# Confirm with 'yes' when prompted
```

### Step 5: Verify Resources
```bash
# Check outputs
terraform output

# View specific output
terraform output instance_public_ip

# List all resources
terraform state list
```

## 🔧 Example 2: Multi-tier Infrastructure

### Scenario: Create VPC with Public and Private Subnets

Create a complete VPC infrastructure with public and private subnets, NAT gateway, and EC2 instances.

### Step 1: Create VPC Configuration
```hcl
# vpc.tf
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = "main-vpc"
  }
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "main-igw"
  }
}

resource "aws_subnet" "public" {
  count = 2
  
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.${count.index + 1}.0/24"
  availability_zone        = data.aws_availability_zones.available.names[count.index]
  map_public_ip_on_launch = true
  
  tags = {
    Name = "public-subnet-${count.index + 1}"
  }
}

resource "aws_subnet" "private" {
  count = 2
  
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index + 10}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]
  
  tags = {
    Name = "private-subnet-${count.index + 1}"
  }
}

resource "aws_eip" "nat" {
  count = 2
  
  domain = "vpc"
  
  tags = {
    Name = "nat-eip-${count.index + 1}"
  }
}

resource "aws_nat_gateway" "main" {
  count = 2
  
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id
  
  tags = {
    Name = "nat-gateway-${count.index + 1}"
  }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
  
  tags = {
    Name = "public-rt"
  }
}

resource "aws_route_table" "private" {
  count = 2
  
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main[count.index].id
  }
  
  tags = {
    Name = "private-rt-${count.index + 1}"
  }
}

resource "aws_route_table_association" "public" {
  count = 2
  
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "private" {
  count = 2
  
  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private[count.index].id
}
```

### Step 2: Create EC2 Instances
```hcl
# instances.tf
resource "aws_instance" "web" {
  count = 2
  
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.public[count.index].id
  
  vpc_security_group_ids = [aws_security_group.web.id]
  
  tags = {
    Name = "web-server-${count.index + 1}"
  }
}

resource "aws_instance" "app" {
  count = 2
  
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.private[count.index].id
  
  vpc_security_group_ids = [aws_security_group.app.id]
  
  tags = {
    Name = "app-server-${count.index + 1}"
  }
}
```

### Step 3: Deploy Infrastructure
```bash
# Initialize
terraform init

# Plan
terraform plan

# Apply
terraform apply
```

## 🚀 Example 3: Using Modules

### Scenario: Reusable Infrastructure Components

Create reusable modules for common infrastructure patterns.

### Step 1: Create Module Structure
```bash
# Directory structure
modules/
  vpc/
    main.tf
    variables.tf
    outputs.tf
  ec2/
    main.tf
    variables.tf
    outputs.tf
```

### Step 2: Create VPC Module
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
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name        = "${var.environment}-vpc"
    Environment = var.environment
  }
}

# modules/vpc/outputs.tf
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "vpc_cidr" {
  description = "CIDR block of the VPC"
  value       = aws_vpc.main.cidr_block
}
```

### Step 3: Use Module
```hcl
# main.tf
module "vpc" {
  source = "./modules/vpc"
  
  vpc_cidr    = "10.0.0.0/16"
  environment = "production"
}

# Reference module output
resource "aws_subnet" "public" {
  vpc_id     = module.vpc.vpc_id
  cidr_block = "10.0.1.0/24"
}
```

## 🎯 Example 4: Workspace Management

### Scenario: Environment-specific Deployments

Use Terraform workspaces to manage multiple environments.

### Step 1: Create Workspaces
```bash
# Create production workspace
terraform workspace new production

# Create staging workspace
terraform workspace new staging

# List workspaces
terraform workspace list

# Switch workspace
terraform workspace select production
```

### Step 2: Use Workspace in Configuration
```hcl
# main.tf
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = terraform.workspace == "production" ? "t3.large" : "t2.micro"
  
  tags = {
    Name        = "web-${terraform.workspace}"
    Environment = terraform.workspace
  }
}

# Use workspace-specific variables
variable "instance_count" {
  type = map(number)
  default = {
    production = 5
    staging    = 2
    dev        = 1
  }
}

resource "aws_instance" "web" {
  count = var.instance_count[terraform.workspace]
  
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
}
```

## 🔄 Example 5: State Management

### Scenario: Manage and Inspect State

Work with Terraform state to inspect and manage resources.

### Step 1: Inspect State
```bash
# List all resources in state
terraform state list

# Show specific resource
terraform state show aws_instance.web

# Show all resources
terraform show
```

### Step 2: Move Resources
```bash
# Move resource in state
terraform state mv aws_instance.web aws_instance.web_new

# Move resource to different state file
terraform state mv -state-out=other.tfstate aws_instance.web aws_instance.web
```

### Step 3: Remove Resources
```bash
# Remove resource from state (doesn't destroy)
terraform state rm aws_instance.old

# Import existing resource
terraform import aws_instance.web i-1234567890abcdef0
```

## 📊 Monitoring and Debugging

### Check Terraform Status
```bash
# Validate configuration
terraform validate

# Format code
terraform fmt

# Check plan
terraform plan -detailed-exitcode

# Show graph
terraform graph | dot -Tsvg > graph.svg
```

### Debug Issues
```bash
# Enable debug logging
export TF_LOG=DEBUG
terraform apply

# Log to file
export TF_LOG_PATH=./terraform.log
terraform apply
```

## 🎯 Common Usage Patterns

### 1. **Infrastructure Provisioning**
```bash
# 1. Write configuration
# 2. Initialize: terraform init
# 3. Plan: terraform plan
# 4. Apply: terraform apply
# 5. Verify: terraform output
```

### 2. **Multi-environment Management**
```bash
# 1. Create workspaces
terraform workspace new production

# 2. Use workspace-specific configs
# 3. Deploy to each environment
terraform workspace select production
terraform apply
```

### 3. **Infrastructure Updates**
```bash
# 1. Modify configuration
# 2. Plan changes
terraform plan

# 3. Apply updates
terraform apply

# 4. Review changes
terraform show
```

---

*Terraform is now integrated into your infrastructure workflow! 🎉*