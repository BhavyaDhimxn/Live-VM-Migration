# Hopsworks — Configuration

## ⚙️ Configuration Overview

Hopsworks configuration involves setting up API keys, projects, feature stores, and various integration settings to customize the platform for your specific needs.

## 📁 Configuration Files

### 1. **Hopsworks Configuration** (`~/.hopsworksrc`)
```ini
[hopsworks]
api_key = your-api-key-here
project = my_project
host = app.hopsworks.ai
port = 443
```

### 2. **Environment Variables**
```bash
# Set API key
export HOPSWORKS_API_KEY="your-api-key-here"

# Set project
export HOPSWORKS_PROJECT="my_project"

# Set host
export HOPSWORKS_HOST="app.hopsworks.ai"
```

## 🔧 Basic Configuration

### 1. **API Key Configuration**
```python
# Method 1: Environment variable
import os
import hopsworks

project = hopsworks.login(
    api_key_value=os.getenv("HOPSWORKS_API_KEY"),
    project="my_project"
)

# Method 2: Direct configuration
project = hopsworks.login(
    api_key_value="your-api-key-here",
    project="my_project"
)

# Method 3: Configuration file
project = hopsworks.login()  # Reads from ~/.hopsworksrc
```

### 2. **Project Configuration**
```python
# Set default project
import hopsworks

project = hopsworks.login(
    api_key_value="your-api-key",
    project="my_project"
)

# Get project information
print(f"Project: {project.name}")
print(f"Owner: {project.owner}")
```

### 3. **Feature Store Configuration**
```python
# Configure feature store
import hsfs

connection = hsfs.connection()
fs = connection.get_feature_store("my_feature_store")

# Configure feature store settings
fs.settings.enable_online_feature_store = True
fs.settings.online_feature_store_type = "redis"
```

## ☁️ Cloud Provider Configuration

### 1. **AWS Configuration**
```python
# Configure AWS integration
import hopsworks

project = hopsworks.login(
    api_key_value="your-api-key",
    project="my_project"
)

# Configure AWS storage
project.storage.configure_aws(
    region="us-west-2",
    bucket="my-hopsworks-bucket"
)
```

### 2. **Google Cloud Configuration**
```python
# Configure GCP integration
import hopsworks

project = hopsworks.login(
    api_key_value="your-api-key",
    project="my_project"
)

# Configure GCP storage
project.storage.configure_gcp(
    project_id="my-project-id",
    bucket="my-hopsworks-bucket"
)
```

### 3. **Azure Configuration**
```python
# Configure Azure integration
import hopsworks

project = hopsworks.login(
    api_key_value="your-api-key",
    project="my_project"
)

# Configure Azure storage
project.storage.configure_azure(
    account_name="my-account",
    container="my-container"
)
```

## 🔐 Security Configuration

### 1. **Authentication Configuration**
```python
# Secure API key handling
import os
from getpass import getpass
import hopsworks

# Get API key securely
api_key = os.getenv("HOPSWORKS_API_KEY")
if not api_key:
    api_key = getpass("Enter your Hopsworks API key: ")

# Initialize connection
project = hopsworks.login(
    api_key_value=api_key,
    project="my_project"
)
```

### 2. **Access Control**
```python
# Configure project access
import hopsworks

project = hopsworks.login(
    api_key_value="your-api-key",
    project="my_project"
)

# Set project permissions
project.set_permissions(
    users=["user1@example.com", "user2@example.com"],
    role="data_scientist"
)
```

## 🔄 Advanced Configuration

### 1. **Feature Store Configuration**
```python
# Advanced feature store configuration
import hsfs

connection = hsfs.connection()
fs = connection.get_feature_store("my_feature_store")

# Configure online feature store
fs.settings.enable_online_feature_store = True
fs.settings.online_feature_store_type = "redis"
fs.settings.online_feature_store_host = "localhost"
fs.settings.online_feature_store_port = 6379

# Configure offline feature store
fs.settings.offline_feature_store_type = "hive"
fs.settings.offline_feature_store_database = "feature_store"
```

### 2. **Model Registry Configuration**
```python
# Configure model registry
import hsml

mr = hsml.connection().get_model_registry()

# Configure registry settings
mr.settings.model_registry_path = "/Models"
mr.settings.experiments_path = "/Experiments"
```

### 3. **Spark Configuration**
```python
# Configure Spark for Hopsworks
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("Hopsworks") \
    .config("spark.sql.warehouse.dir", "/user/hive/warehouse") \
    .config("spark.executor.memory", "4g") \
    .config("spark.executor.cores", "2") \
    .getOrCreate()
```

## 🐛 Common Configuration Issues

### Issue 1: Invalid API Key
```bash
# Error: Invalid API key
# Solution: Check API key
curl -H "Authorization: Bearer $HOPSWORKS_API_KEY" \
     https://app.hopsworks.ai/api/projects
```

### Issue 2: Project Not Found
```bash
# Error: Project not found
# Solution: Check project name
# List available projects
python -c "
import hopsworks
project = hopsworks.login(api_key_value='your-key')
print([p.name for p in project.list_projects()])
"
```

### Issue 3: Feature Store Access Denied
```bash
# Error: Feature store access denied
# Solution: Check feature store permissions
# Verify feature store access
python -c "
import hsfs
connection = hsfs.connection()
fs = connection.get_feature_store('my_feature_store')
print(f'Feature store: {fs.name}')
"
```

### Issue 4: Connection Issues
```bash
# Error: Cannot connect to Hopsworks
# Solution: Check network connectivity
ping app.hopsworks.ai
curl -I https://app.hopsworks.ai/
```

## ✅ Configuration Validation

### Test Configuration
```python
# test_configuration.py
import hopsworks
import hsfs
import hsml

# Test API connection
project = hopsworks.login(
    api_key_value="your-api-key",
    project="my_project"
)
print(f"Connected to project: {project.name}")

# Test feature store
connection = hsfs.connection()
fs = connection.get_feature_store("my_feature_store")
print(f"Feature store: {fs.name}")

# Test model registry
mr = hsml.connection().get_model_registry()
print("Model registry connected")
```

### Configuration Checklist
- [ ] API key configured and valid
- [ ] Project set and accessible
- [ ] Feature store configured
- [ ] Model registry configured
- [ ] Cloud provider configured (if needed)
- [ ] Spark configured (if needed)
- [ ] Authentication configured
- [ ] Access control configured

## 🔧 Configuration Best Practices

1. **🔐 Security**: Use environment variables for API keys
2. **📊 Organization**: Use consistent project and feature store naming
3. **🔄 Automation**: Configure for automated workflows
4. **☁️ Integration**: Set up cloud integrations for scale
5. **📝 Documentation**: Document all configuration changes
6. **🧪 Testing**: Test configuration in development first

---

*Your Hopsworks configuration is now optimized for your MLOps workflow! 🎯*