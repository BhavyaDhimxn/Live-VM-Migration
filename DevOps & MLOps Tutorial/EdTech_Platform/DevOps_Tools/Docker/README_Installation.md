# Docker — Installation

## 🚀 Installation Methods

Docker can be installed on various operating systems using multiple methods.

## 🐳 Method 1: Docker Desktop (Recommended)

### macOS
```bash
# Download Docker Desktop for Mac
# https://www.docker.com/products/docker-desktop

# Or install via Homebrew
brew install --cask docker

# Start Docker Desktop
open -a Docker
```

### Windows
```bash
# Download Docker Desktop for Windows
# https://www.docker.com/products/docker-desktop

# Or install via Chocolatey
choco install docker-desktop

# Or install via Winget
winget install Docker.DockerDesktop
```

## 🐧 Method 2: Linux Installation

### Ubuntu/Debian
```bash
# Update package index
sudo apt-get update

# Install prerequisites
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Add Docker's official GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Set up repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Start Docker
sudo systemctl start docker
sudo systemctl enable docker
```

### CentOS/RHEL
```bash
# Install required packages
sudo yum install -y yum-utils

# Add Docker repository
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

# Install Docker Engine
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Start Docker
sudo systemctl start docker
sudo systemctl enable docker
```

## 🐳 Method 3: Docker Compose

### Installation
```bash
# Docker Compose is included with Docker Desktop
# For Linux, install separately:

# Download latest version
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Make executable
sudo chmod +x /usr/local/bin/docker-compose

# Verify installation
docker-compose --version
```

## 📋 Prerequisites

### System Requirements
- **OS**: Linux, macOS, or Windows
- **Memory**: 4GB+ RAM (8GB+ recommended)
- **Storage**: 20GB+ free space
- **CPU**: 64-bit processor with virtualization support

### Required Tools
```bash
# Check system requirements
uname -a  # Check OS and architecture

# Check virtualization support (Linux)
grep -E 'vmx|svm' /proc/cpuinfo  # Should show output
```

## ⚙️ Installation Steps

### Step 1: Install Docker
```bash
# Choose appropriate method for your OS
# macOS/Windows: Docker Desktop
# Linux: Follow distribution-specific instructions
```

### Step 2: Verify Installation
```bash
# Check Docker version
docker --version

# Check Docker Compose version
docker-compose --version

# Run test container
docker run hello-world
```

### Step 3: Configure Docker (Linux)
```bash
# Add user to docker group (avoid sudo)
sudo usermod -aG docker $USER

# Log out and log back in for changes to take effect
# Or use: newgrp docker

# Verify non-root access
docker run hello-world
```

## 🔧 Post-Installation Setup

### 1. Configure Docker Daemon
```bash
# Edit daemon configuration
sudo nano /etc/docker/daemon.json

# Example configuration:
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "storage-driver": "overlay2"
}

# Restart Docker
sudo systemctl restart docker
```

### 2. Configure Docker Registry
```bash
# Login to Docker Hub
docker login

# Or configure private registry
docker login registry.example.com
```

### 3. Set Up Docker Compose
```bash
# Create docker-compose.yml
cat > docker-compose.yml << EOF
version: '3.8'
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
EOF

# Test Docker Compose
docker-compose up -d
```

## ✅ Verification

### Test Docker Installation
```bash
# Check Docker version
docker --version

# Expected output: Docker version 24.x.x
```

### Test Basic Functionality
```bash
# Run test container
docker run hello-world

# Expected output: Hello from Docker!
```

### Test Docker Compose
```bash
# Test Docker Compose
docker-compose --version

# Run sample compose file
docker-compose up -d
docker-compose ps
docker-compose down
```

## 🐛 Common Installation Issues

### Issue 1: Permission Denied
```bash
# Error: Permission denied while trying to connect to Docker daemon
# Solution: Add user to docker group
sudo usermod -aG docker $USER
newgrp docker
```

### Issue 2: Docker Daemon Not Running
```bash
# Error: Cannot connect to Docker daemon
# Solution: Start Docker service
sudo systemctl start docker
sudo systemctl enable docker
```

### Issue 3: Virtualization Not Enabled
```bash
# Error: Hardware assisted virtualization not available
# Solution: Enable virtualization in BIOS
# Check: grep -E 'vmx|svm' /proc/cpuinfo
```

## 🚀 Next Steps

After successful installation:

1. **📖 Read**: [Configuration Guide](README_Configuration.md)
2. **🛠️ Setup**: Create your first Dockerfile
3. **📊 Practice**: Build your first container
4. **🔄 Learn**: Explore Docker Compose

---

*Docker is now ready for containerization! 🎉*