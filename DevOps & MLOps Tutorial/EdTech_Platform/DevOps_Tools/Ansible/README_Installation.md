# Ansible — Installation

## 🚀 Installation Methods

Ansible can be installed using multiple methods depending on your operating system and preferences.

## 📦 Method 1: pip (Recommended)

### Basic Installation
```bash
# Install Ansible using pip
pip install ansible

# Install specific version
pip install ansible==8.7.0

# Install with extras
pip install ansible[azure]    # Azure modules
pip install ansible[aws]       # AWS modules
pip install ansible[gcp]       # GCP modules
pip install ansible[docker]    # Docker modules
pip install ansible[all]       # All modules
```

### Virtual Environment Installation
```bash
# Create virtual environment
python3 -m venv ansible-env

# Activate virtual environment
source ansible-env/bin/activate  # Linux/macOS
# or
ansible-env\Scripts\activate    # Windows

# Install Ansible
pip install ansible

# Verify installation
ansible --version
```

## 🍎 Method 2: macOS

### Homebrew
```bash
# Install Ansible using Homebrew
brew install ansible

# Verify installation
ansible --version

# Update Ansible
brew upgrade ansible
```

### MacPorts
```bash
# Install Ansible using MacPorts
sudo port install ansible

# Verify installation
ansible --version
```

## 🐧 Method 3: Linux

### Ubuntu/Debian
```bash
# Update package index
sudo apt-get update

# Install software properties common
sudo apt-get install -y software-properties-common

# Add Ansible repository
sudo apt-add-repository --yes --update ppa:ansible/ansible

# Install Ansible
sudo apt-get install -y ansible

# Verify installation
ansible --version
```

### CentOS/RHEL
```bash
# Install EPEL repository
sudo yum install -y epel-release

# Install Ansible
sudo yum install -y ansible

# Verify installation
ansible --version
```

### Fedora
```bash
# Install Ansible
sudo dnf install -y ansible

# Verify installation
ansible --version
```

### Arch Linux
```bash
# Install Ansible
sudo pacman -S ansible

# Verify installation
ansible --version
```

## 🪟 Method 4: Windows

### Using WSL (Windows Subsystem for Linux)
```bash
# Install WSL
wsl --install

# In WSL, install Ansible
sudo apt-get update
sudo apt-get install -y ansible
```

### Using pip on Windows
```bash
# Install Python
# Download from python.org

# Install pip
python -m ensurepip --upgrade

# Install Ansible
pip install ansible

# Note: Some modules may not work on Windows
```

## 🐳 Method 5: Docker

### Run Ansible in Docker
```bash
# Run Ansible via Docker
docker run --rm -v $(pwd):/ansible -w /ansible williamyeh/ansible:latest ansible --version

# Create alias for convenience
alias ansible='docker run --rm -v $(pwd):/ansible -w /ansible williamyeh/ansible:latest ansible'

# Use Ansible
ansible all -m ping
```

### Docker Compose
```yaml
# docker-compose.yml
version: '3.8'
services:
  ansible:
    image: williamyeh/ansible:latest
    volumes:
      - .:/ansible
    working_dir: /ansible
    command: ansible --version
```

## 📋 Prerequisites

### System Requirements
- **Python**: 3.8+ (Python 2.7 is deprecated)
- **Memory**: 512MB+ RAM
- **Storage**: 100MB+ free space
- **Network**: Internet access (for module downloads)
- **Operating System**: Linux, macOS, or Windows (via WSL)

### Required Tools
```bash
# Python
python3 --version

# pip
pip3 --version

# SSH (for remote connections)
ssh --version

# Git (optional, for version control)
git --version
```

## ⚙️ Installation Steps

### Step 1: Install Python
```bash
# Check Python version
python3 --version

# Should be Python 3.8 or higher
# If not installed, install Python 3.8+
```

### Step 2: Install pip
```bash
# Check pip version
pip3 --version

# Install pip if needed
python3 -m ensurepip --upgrade
```

### Step 3: Install Ansible
```bash
# Choose installation method
# pip (recommended):
pip3 install ansible

# Or package manager:
# macOS: brew install ansible
# Ubuntu: sudo apt-get install ansible
# CentOS: sudo yum install ansible
```

### Step 4: Verify Installation
```bash
# Check Ansible version
ansible --version

# Expected output:
# ansible [core 2.15.0]
#   config file = None
#   configured module search path = ['/home/user/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
#   ansible python module location = /usr/local/lib/python3.10/site-packages/ansible
#   ansible collection location = /home/user/.ansible/collections:/usr/share/ansible/collections
#   executable location = /usr/local/bin/ansible
#   python version = 3.10.0
```

## ✅ Verification

### Test Ansible Installation
```bash
# Check Ansible version
ansible --version

# Check Ansible configuration
ansible-config view

# List available modules
ansible-doc -l | head -20

# Test ping module
ansible localhost -m ping
```

### Test Remote Connection
```bash
# Create inventory file
cat > inventory.ini <<EOF
[servers]
server1 ansible_host=192.168.1.10
server2 ansible_host=192.168.1.11

[servers:vars]
ansible_user=admin
ansible_ssh_private_key_file=~/.ssh/id_rsa
EOF

# Test connection
ansible all -i inventory.ini -m ping
```

## 🔧 Post-Installation Configuration

### Configure Ansible
```bash
# Create Ansible configuration directory
mkdir -p ~/.ansible

# Create ansible.cfg
cat > ~/.ansible/ansible.cfg <<EOF
[defaults]
inventory = ~/.ansible/inventory.ini
remote_user = ansible
private_key_file = ~/.ssh/id_rsa
host_key_checking = False
retry_files_enabled = False
roles_path = ~/.ansible/roles
EOF
```

### Install Ansible Collections
```bash
# Install community collections
ansible-galaxy collection install community.general
ansible-galaxy collection install community.docker
ansible-galaxy collection install amazon.aws
ansible-galaxy collection install azure.azcollection
ansible-galaxy collection install google.cloud
```

### Install Ansible Roles
```bash
# Install roles from Ansible Galaxy
ansible-galaxy install geerlingguy.nginx
ansible-galaxy install geerlingguy.docker
ansible-galaxy install geerlingguy.mysql

# List installed roles
ansible-galaxy list
```

## 🐛 Troubleshooting

### Issue 1: Command Not Found
```bash
# Problem: ansible command not found
# Solution: Add to PATH
export PATH=$PATH:~/.local/bin

# Or install system-wide
sudo pip3 install ansible
```

### Issue 2: Python Version
```bash
# Problem: Python version too old
# Solution: Install Python 3.8+
# macOS: brew install python@3.10
# Ubuntu: sudo apt-get install python3.10
```

### Issue 3: Permission Denied
```bash
# Problem: Permission denied
# Solution: Use virtual environment or install with --user
pip3 install --user ansible
```

## 📚 Next Steps

After installation:
1. **Create Inventory**: Set up inventory file
2. **Configure SSH**: Set up SSH keys for remote access
3. **Create Playbooks**: Start writing playbooks
4. **Install Collections**: Install required collections
5. **Set Up Roles**: Organize with roles

---

*Ansible is ready for automation! 🎉*