# DataRobot — Configuration

## ⚙️ Configuration Overview

DataRobot configuration involves setting up API tokens, endpoints, project settings, and various integration settings to customize the platform for your specific needs.

## 📁 Configuration Files

### 1. **DataRobot Configuration** (`~/.datarobotrc`)
```ini
[datarobot]
api_token = your-api-token
endpoint = https://app.datarobot.com/api/v2
default_project = my-project
```

### 2. **Environment Variables**
```bash
# Set API token
export DATAROBOT_API_TOKEN="your-api-token"

# Set endpoint
export DATAROBOT_ENDPOINT="https://app.datarobot.com/api/v2"

# Set default project
export DATAROBOT_DEFAULT_PROJECT="my-project"
```

## 🔧 Basic Configuration

### 1. **API Token Configuration**
```python
# Method 1: Environment variable
import os
import datarobot as dr

client = dr.Client(
    token=os.getenv("DATAROBOT_API_TOKEN"),
    endpoint=os.getenv("DATAROBOT_ENDPOINT", "https://app.datarobot.com/api/v2")
)

# Method 2: Direct configuration
client = dr.Client(
    token="your-api-token",
    endpoint="https://app.datarobot.com/api/v2"
)

# Method 3: Configuration file
client = dr.Client()  # Reads from ~/.datarobotrc
```

### 2. **Project Configuration**
```python
# Configure project settings
import datarobot as dr

# Create project with configuration
project = dr.Project.create(
    project_name="my-project",
    sourcedata="data/train.csv",
    max_wait=3600  # Wait up to 1 hour for autopilot
)

# Configure autopilot mode
project.set_target(
    target="target_column",
    mode=dr.AUTOPILOT_MODE.FULL_AUTO,
    worker_count=-1  # Use all available workers
)
```

### 3. **Model Configuration**
```python
# Configure model training
project.set_target(
    target="target_column",
    mode=dr.AUTOPILOT_MODE.FULL_AUTO,
    advanced_options={
        "blend_best_models": True,
        "prepare_model_for_deployment": True
    }
)
```

## ☁️ Cloud Provider Configuration

### 1. **AWS Configuration**
```python
# Configure AWS integration
import datarobot as dr

# Set AWS credentials
os.environ["AWS_ACCESS_KEY_ID"] = "your-access-key"
os.environ["AWS_SECRET_ACCESS_KEY"] = "your-secret-key"
os.environ["AWS_DEFAULT_REGION"] = "us-west-2"

# Use S3 data source
project = dr.Project.create(
    project_name="aws-project",
    sourcedata="s3://my-bucket/data/train.csv"
)
```

### 2. **Google Cloud Configuration**
```python
# Configure GCP integration
import datarobot as dr

# Set GCP credentials
os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = "/path/to/credentials.json"

# Use GCS data source
project = dr.Project.create(
    project_name="gcp-project",
    sourcedata="gs://my-bucket/data/train.csv"
)
```

### 3. **Azure Configuration**
```python
# Configure Azure integration
import datarobot as dr

# Set Azure credentials
os.environ["AZURE_STORAGE_CONNECTION_STRING"] = "DefaultEndpointsProtocol=https;..."

# Use Azure Blob data source
project = dr.Project.create(
    project_name="azure-project",
    sourcedata="wasbs://container@account.blob.core.windows.net/data/train.csv"
)
```

## 🔐 Security Configuration

### 1. **API Token Management**
```python
# Secure API token handling
import os
from getpass import getpass
import datarobot as dr

# Get API token securely
api_token = os.getenv("DATAROBOT_API_TOKEN")
if not api_token:
    api_token = getpass("Enter your DataRobot API token: ")

# Initialize client
client = dr.Client(
    token=api_token,
    endpoint="https://app.datarobot.com/api/v2"
)
```

### 2. **Access Control**
```python
# Configure project access
import datarobot as dr

# Get project
project = dr.Project.get(project_id)

# Set project permissions
project.set_permissions(
    users=["user1@example.com", "user2@example.com"],
    roles=["project_owner", "project_editor"]
)
```

## 🔄 Advanced Configuration

### 1. **Autopilot Configuration**
```python
# Configure autopilot settings
project.set_target(
    target="target_column",
    mode=dr.AUTOPILOT_MODE.FULL_AUTO,
    advanced_options={
        "blend_best_models": True,
        "prepare_model_for_deployment": True,
        "scoring_code_only": False,
        "seed": 42
    }
)
```

### 2. **Feature Engineering Configuration**
```python
# Configure feature engineering
project.set_target(
    target="target_column",
    feature_engineering_options={
        "enable_feature_discovery": True,
        "max_feature_derivation_time": 3600
    }
)
```

### 3. **Deployment Configuration**
```python
# Configure deployment settings
deployment = dr.Deployment.create_from_learning_model(
    model_id=model.id,
    label="production-deployment",
    description="Production deployment",
    default_prediction_server_id=server.id,
    importance_threshold=0.8
)
```

## 🐛 Common Configuration Issues

### Issue 1: Invalid API Token
```bash
# Error: Invalid API token
# Solution: Check API token
curl -H "Authorization: Bearer $DATAROBOT_API_TOKEN" \
     https://app.datarobot.com/api/v2/projects
```

### Issue 2: Project Not Found
```bash
# Error: Project not found
# Solution: Check project ID
python -c "
import datarobot as dr
projects = dr.Project.list()
print([p.id for p in projects])
"
```

### Issue 3: Data Source Access Denied
```bash
# Error: Access denied to data source
# Solution: Check data source credentials
# For S3:
aws s3 ls s3://my-bucket/data/

# For GCS:
gsutil ls gs://my-bucket/data/
```

## ✅ Configuration Validation

### Test Configuration
```python
# test_configuration.py
import datarobot as dr

# Test API connection
client = dr.Client(
    token="your-api-token",
    endpoint="https://app.datarobot.com/api/v2"
)

# Test project creation
project = dr.Project.create(
    project_name="test-project",
    sourcedata="data/test.csv"
)

print(f"Project created: {project.id}")
```

### Configuration Checklist
- [ ] API token configured and valid
- [ ] Endpoint configured correctly
- [ ] Project settings configured
- [ ] Cloud provider configured (if needed)
- [ ] Deployment settings configured
- [ ] Security settings configured

## 🔧 Configuration Best Practices

1. **🔐 Security**: Use environment variables for API tokens
2. **📊 Organization**: Use consistent project naming
3. **🔄 Automation**: Configure for automated workflows
4. **☁️ Integration**: Set up cloud integrations
5. **📝 Documentation**: Document all configuration changes

---

*Your DataRobot configuration is now optimized for your ML workflow! 🎯*