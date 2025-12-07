# Terraform — Installation

## 🚀 Installation Methods

Terraform can be installed using multiple methods depending on your operating system and preferences.

## 📦 Method 1: Package Managers (Recommended)

### macOS
```bash
# Using Homebrew (Recommended)
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

# Verify installation
terraform version

# Update Terraform
brew upgrade hashicorp/tap/terraform
```

### Linux (Ubuntu/Debian)
```bash
# Add HashiCorp GPG key
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -

# Add HashiCorp repository
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"

# Update and install
sudo apt-get update
sudo apt-get install terraform

# Verify installation
terraform version
```

### Linux (CentOS/RHEL)
```bash
# Add HashiCorp repository
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo

# Install Terraform
sudo yum -y install terraform

# Verify installation
terraform version
```

### Windows
```bash
# Using Chocolatey
choco install terraform

# Or using Scoop
scoop bucket add hashicorp
scoop install terraform

# Verify installation
terraform version
```

## 🐧 Method 2: Manual Binary Installation

### Download and Install
```bash
# Download latest version
# Visit: https://www.terraform.io/downloads

# For Linux
wget https://releases.hashicorp.com/terraform/1.6.0/terraform_1.6.0_linux_amd64.zip

# Extract
unzip terraform_1.6.0_linux_amd64.zip

# Move to PATH
sudo mv terraform /usr/local/bin/

# Verify installation
terraform version
```

### macOS Manual Installation
```bash
# Download
wget https://releases.hashicorp.com/terraform/1.6.0/terraform_1.6.0_darwin_amd64.zip

# Extract
unzip terraform_1.6.0_darwin_amd64.zip

# Move to PATH
sudo mv terraform /usr/local/bin/

# Verify installation
terraform version
```

## 🐳 Method 3: Docker

### Run Terraform in Docker
```bash
# Run Terraform via Docker
docker run --rm -v $(pwd):/workspace -w /workspace hashicorp/terraform:latest version

# Create alias for convenience
alias terraform='docker run --rm -v $(pwd):/workspace -w /workspace hashicorp/terraform:latest'

# Use Terraform
terraform init
terraform plan
terraform apply
```

### Docker Compose
```yaml
# docker-compose.yml
version: '3.8'
services:
  terraform:
    image: hashicorp/terraform:latest
    volumes:
      - .:/workspace
    working_dir: /workspace
    command: version
```

## ☁️ Method 4: Cloud Shell

### AWS CloudShell
```bash
# Terraform is pre-installed in AWS CloudShell
terraform version

# Start using immediately
terraform init
```

### Google Cloud Shell
```bash
# Install Terraform in Cloud Shell
wget https://releases.hashicorp.com/terraform/1.6.0/terraform_1.6.0_linux_amd64.zip
unzip terraform_1.6.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/
```

### Azure Cloud Shell
```bash
# Terraform is available in Azure Cloud Shell
terraform version
```

## 📋 Prerequisites

### System Requirements
- **Operating System**: Linux, macOS, or Windows
- **Architecture**: AMD64 (x86_64)
- **Memory**: 512MB+ RAM (2GB+ recommended)
- **Storage**: 100MB+ free space
- **Network**: Internet access for provider downloads

### Required Tools
```bash
# Git (for version control)
git --version

# Text Editor (VS Code, Vim, etc.)
# For VS Code, install HashiCorp Terraform extension
```

## ⚙️ Installation Steps

### Step 1: Choose Installation Method
```bash
# Choose based on your OS:
# - macOS: Homebrew
# - Linux: Package manager or manual
# - Windows: Chocolatey or manual
# - Docker: For containerized environments
```

### Step 2: Install Terraform
```bash
# Follow method-specific instructions above
# Example for macOS:
brew install hashicorp/tap/terraform
```

### Step 3: Verify Installation
```bash
# Check Terraform version
terraform version

# Expected output:
# Terraform v1.6.0
# on linux_amd64
```

### Step 4: Install Terraform Providers
```bash
# Providers are installed automatically on first use
# Or pre-install common providers:
terraform init
```

## ✅ Verification

### Test Terraform Installation
```bash
# Check version
terraform version

# Check available commands
terraform help

# Initialize a test project
mkdir terraform-test
cd terraform-test

# Create basic configuration
cat > main.tf <<EOF
terraform {
  required_version = ">= 1.0"
}

provider "aws" {
  region = "us-west-2"
}
EOF

# Initialize (will download AWS provider)
terraform init

# Verify providers downloaded
ls -la .terraform/providers/
```

### Verify Provider Installation
```bash
# Check provider versions
terraform providers

# Expected output shows installed providers
```

## 🔧 Post-Installation Configuration

### Configure Terraform CLI
```bash
# Set up autocomplete (bash)
terraform -install-autocomplete

# Reload shell
source ~/.bashrc

# For zsh
echo 'complete -o nospace -C /usr/local/bin/terraform terraform' >> ~/.zshrc
source ~/.zshrc
```

### Configure Backend (Optional)
```hcl
# terraform.tf
terraform {
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "terraform.tfstate"
    region = "us-west-2"
  }
}
```

## 🐛 Troubleshooting

### Issue 1: Command Not Found
```bash
# Problem: terraform command not found
# Solution: Add to PATH
export PATH=$PATH:/usr/local/bin
# Or add to ~/.bashrc or ~/.zshrc
```

### Issue 2: Permission Denied
```bash
# Problem: Permission denied when running terraform
# Solution: Make executable
chmod +x /usr/local/bin/terraform
```

### Issue 3: Provider Download Fails
```bash
# Problem: Cannot download providers
# Solution: Check network connectivity
terraform init -upgrade

# Or use proxy
export HTTP_PROXY=http://proxy.example.com:8080
export HTTPS_PROXY=http://proxy.example.com:8080
```

## 📚 Next Steps

After installation:
1. **Learn Basics**: Start with simple configurations
2. **Set Up Backend**: Configure remote state storage
3. **Install Providers**: Providers install automatically on first use
4. **Configure IDE**: Install Terraform extension for your IDE

---

*Terraform is ready for Infrastructure as Code! 🎉*