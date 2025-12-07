# Jenkins — Installation

## 🚀 Installation Methods

Jenkins can be installed using multiple methods depending on your environment and needs.

## 🐳 Method 1: Docker (Recommended)

### Basic Docker Installation
```bash
# Pull Jenkins image
docker pull jenkins/jenkins:lts

# Run Jenkins container
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts

# Access Jenkins at http://localhost:8080
```

### Docker Compose
```yaml
# docker-compose.yml
version: '3.8'
services:
  jenkins:
    image: jenkins/jenkins:lts
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - jenkins_home:/var/jenkins_home
    environment:
      - JENKINS_OPTS=--httpPort=8080

volumes:
  jenkins_home:
```

## ☕ Method 2: Java WAR File

### Download and Run
```bash
# Download Jenkins WAR file
wget https://get.jenkins.io/war-stable/latest/jenkins.war

# Run Jenkins
java -jar jenkins.war --httpPort=8080

# Access at http://localhost:8080
```

## 🍎 Method 3: macOS Installation

### Homebrew
```bash
# Install Jenkins
brew install jenkins-lts

# Start Jenkins
brew services start jenkins-lts

# Access at http://localhost:8080
```

## 🐧 Method 4: Linux Installation

### Ubuntu/Debian
```bash
# Add Jenkins repository
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# Update and install
sudo apt-get update
sudo apt-get install jenkins

# Start Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

### CentOS/RHEL
```bash
# Add Jenkins repository
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo

sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

# Install Jenkins
sudo yum install jenkins

# Start Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

## 🪟 Method 5: Windows Installation

### Windows Installer
```bash
# Download installer from https://www.jenkins.io/download/
# Run jenkins.msi installer
# Follow installation wizard
# Jenkins will run as Windows service
```

## 📋 Prerequisites

### System Requirements
- **Java**: JDK 11 or JDK 17 (LTS versions)
- **Memory**: 512MB minimum (2GB+ recommended)
- **Storage**: 10GB+ free space
- **Network**: Internet access for plugin installation

### Required Tools
```bash
# Check Java version
java -version  # Should be 11 or 17

# Install Java if needed
# Ubuntu/Debian:
sudo apt-get install openjdk-17-jdk

# macOS:
brew install openjdk@17
```

## ⚙️ Installation Steps

### Step 1: Install Jenkins
```bash
# Choose appropriate method
# Docker: docker run jenkins/jenkins:lts
# Linux: sudo apt-get install jenkins
# macOS: brew install jenkins-lts
```

### Step 2: Access Jenkins
```bash
# Open browser
# http://localhost:8080

# Get initial admin password
# Docker:
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

# Linux:
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### Step 3: Initial Setup
```bash
# 1. Enter admin password
# 2. Install suggested plugins
# 3. Create admin user
# 4. Configure Jenkins URL
# 5. Start using Jenkins
```

## 🔧 Post-Installation Setup

### 1. Install Essential Plugins
```bash
# Via Web UI: Manage Jenkins > Plugins
# Essential plugins:
# - Git Plugin
# - Pipeline Plugin
# - Docker Pipeline Plugin
# - Blue Ocean Plugin
```

### 2. Configure Global Tools
```bash
# Manage Jenkins > Global Tool Configuration
# Configure:
# - JDK installations
# - Git installations
# - Maven/Gradle installations
# - Docker installations
```

### 3. Set Up Credentials
```bash
# Manage Jenkins > Credentials
# Add:
# - Git credentials (SSH keys, tokens)
# - Docker registry credentials
# - Cloud provider credentials
```

## ✅ Verification

### Test Jenkins Installation
```bash
# Check Jenkins status
# Docker:
docker ps | grep jenkins

# Linux:
sudo systemctl status jenkins

# Access web UI
curl http://localhost:8080
```

### Test Basic Functionality
```bash
# Create test job
# 1. New Item > Freestyle project
# 2. Add build step: Execute shell
# 3. Command: echo "Hello Jenkins"
# 4. Save and Build
# 5. Check console output
```

## 🐛 Common Installation Issues

### Issue 1: Port Already in Use
```bash
# Error: Port 8080 already in use
# Solution: Use different port
docker run -p 8081:8080 jenkins/jenkins:lts
# Or: java -jar jenkins.war --httpPort=8081
```

### Issue 2: Java Version Issues
```bash
# Error: Unsupported Java version
# Solution: Install Java 11 or 17
java -version  # Check version
# Update JAVA_HOME if needed
```

### Issue 3: Permission Issues
```bash
# Error: Permission denied
# Solution: Fix permissions
sudo chown -R jenkins:jenkins /var/lib/jenkins
sudo chmod -R 755 /var/lib/jenkins
```

## 🚀 Next Steps

After successful installation:

1. **📖 Read**: [Configuration Guide](README_Configuration.md)
2. **🛠️ Setup**: Configure your first pipeline
3. **📊 Practice**: Create your first Jenkins job
4. **🔄 Learn**: Explore Jenkins pipelines

---

*Jenkins is now ready for CI/CD automation! 🎉*