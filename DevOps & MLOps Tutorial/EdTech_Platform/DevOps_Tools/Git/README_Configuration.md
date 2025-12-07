# Git — Configuration

## ⚙️ Configuration Overview

Git configuration is managed through the `.gitconfig` file and can be set at three levels: system, global, and local.

## 📁 Configuration Files

### 1. **System Configuration** (`/etc/gitconfig`)
```bash
# Applies to all users on the system
git config --system user.name "System User"
git config --system user.email "system@example.com"
```

### 2. **Global Configuration** (`~/.gitconfig`)
```bash
# Applies to all repositories for the current user
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

### 3. **Local Configuration** (`.git/config`)
```bash
# Applies only to the current repository
git config --local user.name "Project Specific Name"
git config --local user.email "project@example.com"
```

## 🔧 Basic Configuration

### 1. **User Identity**
```bash
# Set your name
git config --global user.name "John Doe"

# Set your email
git config --global user.email "john.doe@example.com"

# Verify configuration
git config --global user.name
git config --global user.email
```

### 2. **Default Editor**
```bash
# Set default editor
git config --global core.editor "code --wait"  # VS Code
git config --global core.editor "vim"          # Vim
git config --global core.editor "nano"         # Nano
git config --global core.editor "subl -n -w"   # Sublime Text

# Windows
git config --global core.editor "'C:/Program Files/Notepad++/notepad++.exe' -multiInst -notabbar -nosession -noPlugin"
```

### 3. **Default Branch Name**
```bash
# Set default branch name
git config --global init.defaultBranch main

# Or use 'master' (legacy)
git config --global init.defaultBranch master
```

## 🔐 Authentication Configuration

### 1. **SSH Configuration**
```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"

# Configure SSH for GitHub
cat >> ~/.ssh/config << EOF
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes
EOF

# Test SSH connection
ssh -T git@github.com
```

### 2. **HTTPS Credential Helper**
```bash
# Configure credential helper
git config --global credential.helper store  # Store credentials
git config --global credential.helper cache  # Cache credentials (15 min)

# macOS Keychain
git config --global credential.helper osxkeychain

# Windows Credential Manager
git config --global credential.helper wincred

# Linux (store in file)
git config --global credential.helper store
```

### 3. **Personal Access Token**
```bash
# For GitHub/GitLab, use personal access tokens
# Store token in credential helper
git config --global credential.helper store

# First push will prompt for username and token
git push origin main
# Username: your-username
# Password: your-personal-access-token
```

## 🔄 Advanced Configuration

### 1. **Aliases**
```bash
# Useful Git aliases
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual '!gitk'
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

### 2. **Line Ending Configuration**
```bash
# Auto handle line endings
git config --global core.autocrlf true   # Windows
git config --global core.autocrlf input # Linux/macOS

# Or disable autocrlf
git config --global core.autocrlf false
```

### 3. **Merge Configuration**
```bash
# Set default merge strategy
git config --global merge.tool vimdiff
git config --global merge.conflictstyle diff3

# Set default pull strategy
git config --global pull.rebase false  # Merge (default)
git config --global pull.rebase true   # Rebase
```

### 4. **Color Configuration**
```bash
# Enable colored output
git config --global color.ui true

# Configure specific colors
git config --global color.branch auto
git config --global color.diff auto
git config --global color.status auto
```

## 🔐 Security Configuration

### 1. **GPG Signing**
```bash
# Generate GPG key
gpg --full-generate-key

# List GPG keys
gpg --list-secret-keys --keyid-format LONG

# Configure Git to use GPG
git config --global user.signingkey YOUR_GPG_KEY_ID
git config --global commit.gpgsign true

# Sign tags
git config --global tag.gpgSign true
```

### 2. **Credential Security**
```bash
# Use credential helper securely
git config --global credential.helper osxkeychain  # macOS
git config --global credential.helper wincred      # Windows
git config --global credential.helper store         # Linux (less secure)

# Set credential cache timeout
git config --global credential.helper 'cache --timeout=3600'
```

### 3. **Safe Directory Configuration**
```bash
# Add safe directories (Git 2.35.2+)
git config --global --add safe.directory /path/to/repo
git config --global --add safe.directory '*'
```

## 🔄 Remote Configuration

### 1. **Remote Repository Setup**
```bash
# Add remote repository
git remote add origin https://github.com/user/repo.git

# Add multiple remotes
git remote add upstream https://github.com/original/repo.git

# View remotes
git remote -v

# Update remote URL
git remote set-url origin https://github.com/user/new-repo.git

# Remove remote
git remote remove origin
```

### 2. **Push Configuration**
```bash
# Set default push behavior
git config --global push.default simple
git config --global push.default matching
git config --global push.default current

# Set upstream tracking
git push -u origin main
```

## 🐛 Common Configuration Issues

### Issue 1: Wrong User Identity
```bash
# Error: Commits show wrong name/email
# Solution: Check and update configuration
git config --global user.name
git config --global user.email

# Update if needed
git config --global user.name "Correct Name"
git config --global user.email "correct@example.com"
```

### Issue 2: Credential Issues
```bash
# Error: Authentication failed
# Solution: Clear cached credentials
git credential-cache exit  # Clear cache
git credential-store erase  # Clear stored credentials

# Re-authenticate
git push origin main
```

### Issue 3: Line Ending Issues
```bash
# Error: Line ending conflicts
# Solution: Configure autocrlf
git config --global core.autocrlf true  # Windows
git config --global core.autocrlf input # Linux/macOS
```

## ✅ Configuration Validation

### Test Configuration
```bash
# View all configuration
git config --list

# View global configuration
git config --global --list

# View local configuration
git config --local --list

# Test specific setting
git config user.name
git config user.email
```

### Configuration Checklist
- [ ] User name configured
- [ ] User email configured
- [ ] Default editor set
- [ ] Default branch name set
- [ ] SSH keys configured (if using SSH)
- [ ] Credential helper configured (if using HTTPS)
- [ ] GPG signing configured (optional)
- [ ] Aliases configured (optional)

## 🔧 Configuration Best Practices

1. **🔐 Security**: Use SSH keys or personal access tokens
2. **📝 Identity**: Set correct name and email
3. **🔄 Workflow**: Configure aliases for efficiency
4. **🌍 Compatibility**: Set appropriate line ending handling
5. **📊 Visibility**: Enable colored output for better readability
6. **🛡️ Signing**: Use GPG signing for important commits

---

*Your Git configuration is now optimized for your workflow! 🎯*