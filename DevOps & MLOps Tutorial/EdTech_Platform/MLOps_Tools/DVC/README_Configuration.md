# DVC — Configuration

## ⚙️ Configuration Overview

DVC configuration is managed through multiple files and methods, allowing you to customize behavior for different environments and use cases.

## 📁 Configuration Files

### 1. **Global Configuration** (`~/.dvc/config`)
```ini
[global]
# Global settings for all DVC projects
log_level = info
cache_dir = ~/.cache/dvc
```

### 2. **Project Configuration** (`.dvc/config`)
```ini
[core]
# Project-specific settings
remote = myremote
autostage = true

['remote "myremote"']
url = s3://my-bucket/dvc-storage
region = us-west-2

['remote "local"']
url = /tmp/dvc-storage
```

### 3. **Local Configuration** (`.dvc/config.local`)
```ini
# Local overrides (not committed to Git)
['remote "myremote"']
access_key_id = YOUR_ACCESS_KEY
secret_access_key = YOUR_SECRET_KEY
```

## 🔧 Basic Configuration

### Initialize DVC Configuration
```bash
# Initialize DVC in your project
dvc init

# This creates .dvc/config with default settings
```

### View Current Configuration
```bash
# Show all configuration
dvc config --list

# Show specific section
dvc config --list core
dvc config --list remote
```

## ☁️ Remote Storage Configuration

### AWS S3 Configuration
```bash
# Add S3 remote
dvc remote add -d myremote s3://my-bucket/dvc-storage

# Configure S3 settings
dvc remote modify myremote region us-west-2
dvc remote modify myremote profile my-aws-profile

# Set credentials (use environment variables for security)
export AWS_ACCESS_KEY_ID=your_access_key
export AWS_SECRET_ACCESS_KEY=your_secret_key
```

### Google Cloud Storage Configuration
```bash
# Add GCS remote
dvc remote add -d myremote gs://my-bucket/dvc-storage

# Configure GCS settings
dvc remote modify myremote project_id my-gcp-project
dvc remote modify myremote credentials_path ~/.config/gcloud/credentials.json
```

### Azure Blob Storage Configuration
```bash
# Add Azure remote
dvc remote add -d myremote azure://my-container/dvc-storage

# Configure Azure settings
dvc remote modify myremote account_name my-storage-account
dvc remote modify myremote connection_string "DefaultEndpointsProtocol=https;..."
```

### Local Storage Configuration
```bash
# Add local remote
dvc remote add -d myremote /path/to/local/storage

# Or use relative path
dvc remote add -d myremote ./local-storage
```

## 🗂️ Cache Configuration

### Cache Directory Setup
```bash
# Set custom cache directory
dvc config cache.dir /path/to/cache

# Set cache type (hardlink, symlink, copy)
dvc config cache.type hardlink

# Set cache size limit
dvc config cache.size 10GB
```

### Cache Configuration Examples
```ini
[cache]
# Cache settings
dir = /tmp/dvc-cache
type = hardlink
size = 10GB
```

## 🔐 Security Configuration

### Environment Variables (Recommended)
```bash
# Set credentials via environment variables
export AWS_ACCESS_KEY_ID=your_access_key
export AWS_SECRET_ACCESS_KEY=your_secret_key
export AWS_DEFAULT_REGION=us-west-2

# For Google Cloud
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/credentials.json

# For Azure
export AZURE_STORAGE_CONNECTION_STRING="DefaultEndpointsProtocol=https;..."
```

### Credential Files
```bash
# AWS credentials file (~/.aws/credentials)
[default]
aws_access_key_id = YOUR_ACCESS_KEY
aws_secret_access_key = YOUR_SECRET_KEY

# Google Cloud credentials
gcloud auth application-default login
```

## 📊 Pipeline Configuration

### dvc.yaml Configuration
```yaml
# dvc.yaml - Pipeline definition
stages:
  prepare:
    cmd: python src/prepare.py
    deps:
    - src/prepare.py
    - data/raw
    outs:
    - data/prepared
    params:
    - prepare.split_ratio
    metrics:
    - metrics/prepare.json

  train:
    cmd: python src/train.py
    deps:
    - src/train.py
    - data/prepared
    outs:
    - models/model.pkl
    params:
    - train.epochs
    - train.lr
    metrics:
    - metrics/train.json
```

### params.yaml Configuration
```yaml
# params.yaml - Parameters file
prepare:
  split_ratio: 0.8
  random_seed: 42

train:
  epochs: 100
  lr: 0.001
  batch_size: 32
  optimizer: adam

model:
  architecture: resnet50
  pretrained: true
```

## 🔄 Advanced Configuration

### Autostage Configuration
```bash
# Automatically stage files after dvc add
dvc config core.autostage true

# Disable autostage
dvc config core.autostage false
```

### Logging Configuration
```bash
# Set log level
dvc config core.log_level debug
dvc config core.log_level info
dvc config core.log_level warning
dvc config core.log_level error
```

### Checkpoint Configuration
```bash
# Enable checkpoints for long-running processes
dvc config core.checkpoints true

# Set checkpoint interval
dvc config core.checkpoints_interval 300  # 5 minutes
```

## 🐛 Common Configuration Issues

### Issue 1: Remote Not Found
```bash
# Error: Remote 'myremote' not found
# Solution: Check remote configuration
dvc remote list
dvc remote add -d myremote s3://my-bucket/dvc-storage
```

### Issue 2: Permission Denied
```bash
# Error: Permission denied accessing remote
# Solution: Check credentials and permissions
dvc remote modify myremote access_key_id YOUR_ACCESS_KEY
dvc remote modify myremote secret_access_key YOUR_SECRET_KEY
```

### Issue 3: Cache Directory Issues
```bash
# Error: Cannot create cache directory
# Solution: Set proper cache directory
dvc config cache.dir /path/to/writable/directory
```

### Issue 4: Network Timeout
```bash
# Error: Connection timeout
# Solution: Increase timeout settings
dvc config remote.myremote.timeout 300
```

## ✅ Configuration Validation

### Test Configuration
```bash
# Test remote connection
dvc remote list

# Test cache directory
dvc cache dir

# Test pipeline configuration
dvc dag

# Validate params file
dvc params diff
```

### Configuration Checklist
- [ ] Remote storage configured and accessible
- [ ] Cache directory set and writable
- [ ] Credentials properly configured
- [ ] Pipeline files (dvc.yaml, params.yaml) valid
- [ ] Log level appropriate for your needs
- [ ] Autostage setting matches workflow

## 🔧 Configuration Best Practices

1. **🔐 Security**: Use environment variables for credentials
2. **📁 Organization**: Keep local config in `.dvc/config.local`
3. **🔄 Consistency**: Use same remote across team members
4. **📊 Monitoring**: Set appropriate log levels
5. **⚡ Performance**: Configure cache type based on filesystem
6. **🛡️ Backup**: Regularly backup configuration files

---

*Your DVC configuration is now optimized for your ML workflow! 🎯*