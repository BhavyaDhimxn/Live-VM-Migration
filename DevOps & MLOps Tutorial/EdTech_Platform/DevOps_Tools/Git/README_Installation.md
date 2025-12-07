# Git — Installation

## 🚀 Installation Methods

Git can be installed on various operating systems using multiple methods.

## 🍎 macOS Installation

### Method 1: Homebrew (Recommended)
```bash
# Install Homebrew (if not installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Git
brew install git

# Verify installation
git --version
```

### Method 2: Xcode Command Line Tools
```bash
# Install Xcode Command Line Tools
xcode-select --install

# Git is included with Xcode tools
git --version
```

### Method 3: Official Installer
```bash
# Download from https://git-scm.com/download/mac
# Run the installer package
# Follow installation wizard
```

## 🐧 Linux Installation

### Ubuntu/Debian
```bash
# Update package list
sudo apt-get update

# Install Git
sudo apt-get install git

# Verify installation
git --version
```

### CentOS/RHEL/Fedora
```bash
# Install Git
sudo yum install git
# Or for newer versions:
sudo dnf install git

# Verify installation
git --version
```

### Arch Linux
```bash
# Install Git
sudo pacman -S git

# Verify installation
git --version
```

## 🪟 Windows Installation

### Method 1: Git for Windows (Recommended)
```bash
# Download from https://git-scm.com/download/win
# Run the installer
# Choose options:
# - Use Git from the command line
# - Use the OpenSSL library
# - Checkout Windows-style, commit Unix-style line endings
```

### Method 2: Chocolatey
```bash
# Install Chocolatey (if not installed)
# Then install Git
choco install git

# Verify installation
git --version
```

### Method 3: Winget
```bash
# Install Git using Windows Package Manager
winget install Git.Git

# Verify installation
git --version
```

## 📦 Method 4: Docker

### Git in Docker
```bash
# Pull Git Docker image
docker pull alpine/git

# Run Git commands
docker run -it --rm -v $(pwd):/workspace alpine/git sh

# Or use as alias
alias git='docker run -it --rm -v $(pwd):/workspace alpine/git'
```

## ⚙️ Installation Steps

### Step 1: Install Git
```bash
# Choose appropriate method for your OS
# macOS: brew install git
# Linux: sudo apt-get install git
# Windows: Download from git-scm.com
```

### Step 2: Configure Git
```bash
# Set your name
git config --global user.name "Your Name"

# Set your email
git config --global user.email "your.email@example.com"

# Set default editor
git config --global core.editor "code --wait"  # VS Code
# Or: git config --global core.editor "vim"

# Set default branch name
git config --global init.defaultBranch main
```

### Step 3: Verify Installation
```bash
# Check Git version
git --version

# Check configuration
git config --list

# Test basic commands
git init test-repo
cd test-repo
echo "# Test" > README.md
git add README.md
git commit -m "Initial commit"
cd ..
rm -rf test-repo
```

## 🔧 Post-Installation Setup

### 1. Configure SSH Keys
```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"

# Start SSH agent
eval "$(ssh-agent -s)"

# Add SSH key to agent
ssh-add ~/.ssh/id_ed25519

# Copy public key
cat ~/.ssh/id_ed25519.pub
# Add to GitHub/GitLab/Bitbucket
```

### 2. Configure GPG Signing (Optional)
```bash
# Generate GPG key
gpg --full-generate-key

# List GPG keys
gpg --list-secret-keys --keyid-format LONG

# Configure Git to use GPG
git config --global user.signingkey YOUR_GPG_KEY_ID
git config --global commit.gpgsign true
```

### 3. Set Up Git Aliases
```bash
# Useful aliases
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual '!gitk'
```

## ✅ Verification

### Test Git Installation
```bash
# Check Git version
git --version

# Expected output: git version 2.x.x
```

### Test Basic Functionality
```bash
# Create test repository
mkdir test-git && cd test-git
git init

# Create test file
echo "Hello Git" > test.txt

# Stage and commit
git add test.txt
git commit -m "Test commit"

# Check status
git status

# View log
git log

# Clean up
cd ..
rm -rf test-git
```

### Test Remote Connection
```bash
# Test SSH connection to GitHub
ssh -T git@github.com

# Test HTTPS connection
git ls-remote https://github.com/user/repo.git
```

## 🐛 Common Installation Issues

### Issue 1: Command Not Found
```bash
# Error: git: command not found
# Solution: Add Git to PATH
# macOS/Linux: Usually automatic
# Windows: Check "Add Git to PATH" during installation
```

### Issue 2: Permission Denied
```bash
# Error: Permission denied
# Solution: Use sudo (Linux) or run as administrator (Windows)
sudo apt-get install git  # Linux
```

### Issue 3: Old Version
```bash
# Error: Git version too old
# Solution: Update Git
# macOS:
brew upgrade git

# Linux:
sudo apt-get update && sudo apt-get upgrade git

# Windows: Download latest from git-scm.com
```

### Issue 4: SSH Key Issues
```bash
# Error: Permission denied (publickey)
# Solution: Check SSH key setup
ssh-add -l  # List loaded keys
ssh-add ~/.ssh/id_ed25519  # Add key
```

## 🔍 Troubleshooting Commands

### Check Installation
```bash
# Check Git version
git --version

# Check Git location
which git  # macOS/Linux
where git  # Windows

# Check configuration
git config --list
git config --global --list
```

### Check SSH Setup
```bash
# Test SSH connection
ssh -T git@github.com
ssh -T git@gitlab.com

# Check SSH keys
ls -la ~/.ssh
ssh-add -l
```

## 🚀 Next Steps

After successful installation:

1. **📖 Read**: [Configuration Guide](README_Configuration.md)
2. **🛠️ Setup**: Configure your Git identity
3. **📊 Practice**: Create your first repository
4. **🔄 Learn**: Explore Git workflows

---

*Git is now ready for version control! 🎉*