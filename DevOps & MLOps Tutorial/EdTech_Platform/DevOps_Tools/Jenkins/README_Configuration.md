# Jenkins — Configuration

## ⚙️ Configuration Overview

Jenkins configuration is managed through the web UI, configuration files, and environment variables to customize the server for your specific needs.

## 📁 Configuration Files

### 1. **Jenkins Configuration** (`/var/jenkins_home/config.xml`)
```xml
<!-- Main Jenkins configuration -->
<?xml version='1.0' encoding='UTF-8'?>
<hudson>
    <version>2.414</version>
    <numExecutors>2</numExecutors>
    <mode>NORMAL</mode>
    <useSecurity>true</useSecurity>
    <!-- ... -->
</hudson>
```

### 2. **Environment Variables**
```bash
# Set Jenkins home directory
export JENKINS_HOME=/var/jenkins_home

# Set Jenkins URL
export JENKINS_URL=http://localhost:8080

# Set Java options
export JAVA_OPTS="-Xmx2048m -Xms512m"
```

## 🔧 Basic Configuration

### 1. **System Configuration**
```bash
# Configure via Web UI: Manage Jenkins > Configure System
# Settings:
# - Number of executors
# - Jenkins URL
# - System message
# - Global properties
```

### 2. **Global Tool Configuration**
```bash
# Manage Jenkins > Global Tool Configuration
# Configure:
# - JDK installations
# - Git installations
# - Maven/Gradle installations
# - Docker installations
```

### 3. **Plugin Configuration**
```bash
# Manage Jenkins > Plugins
# Install plugins:
# - Git Plugin
# - Pipeline Plugin
# - Docker Pipeline Plugin
# - Kubernetes Plugin
```

## 🔐 Security Configuration

### 1. **Authentication**
```bash
# Manage Jenkins > Configure Global Security
# Authentication methods:
# - Jenkins' own user database
# - LDAP
# - GitHub OAuth
# - GitLab OAuth
# - SAML
```

### 2. **Authorization**
```bash
# Configure authorization:
# - Matrix-based security
# - Project-based Matrix Authorization
# - Role-Based Strategy
# - GitHub Authorization
```

### 3. **Credential Management**
```bash
# Manage Jenkins > Credentials
# Store credentials:
# - Username/password
# - SSH keys
# - Secret text
# - Certificates
```

## 🔄 Advanced Configuration

### 1. **Pipeline Configuration**
```groovy
// Jenkinsfile configuration
pipeline {
    agent any
    
    options {
        timeout(time: 1, unit: 'HOURS')
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
    
    environment {
        NODE_ENV = 'production'
        DOCKER_REGISTRY = 'registry.example.com'
    }
    
    stages {
        // Pipeline stages
    }
}
```

### 2. **Agent Configuration**
```groovy
// Configure agents
pipeline {
    agent {
        label 'docker-agent'
    }
    // Or
    agent {
        docker {
            image 'maven:3.8-openjdk-17'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
}
```

### 3. **Cloud Configuration**
```bash
# Configure cloud providers
# Manage Jenkins > Configure Clouds
# - AWS EC2
# - Azure VM
# - Google Cloud
# - Kubernetes
```

## 🐛 Common Configuration Issues

### Issue 1: Plugin Installation Failed
```bash
# Error: Plugin installation failed
# Solution: Check network connectivity
# Or install plugins manually
# Download plugin from https://plugins.jenkins.io/
```

### Issue 2: Credential Access Denied
```bash
# Error: Credential access denied
# Solution: Check credential permissions
# Manage Jenkins > Credentials > System > Global credentials
```

## ✅ Configuration Validation

### Test Configuration
```bash
# Check Jenkins configuration
# Via Web UI: Manage Jenkins > System Information

# Test pipeline
# Create test pipeline job
# Run and verify execution
```

---

*Your Jenkins configuration is now optimized for your CI/CD workflow! 🎯*