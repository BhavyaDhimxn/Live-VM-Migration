# CometML — Configuration

## ⚙️ Configuration Overview

CometML configuration involves setting up API keys, workspaces, projects, and various integration settings to customize the platform for your specific needs.

## 📁 Configuration Files

### 1. **CometML Configuration** (`~/.cometrc`)
```ini
[comet]
api_key = your-api-key-here
workspace = your-workspace
project = your-project

[logging]
level = INFO
format = %(asctime)s - %(name)s - %(levelname)s - %(message)s

[experiment]
auto_log_code = true
auto_log_system_info = true
auto_log_graph = true
```

### 2. **Environment Variables**
```bash
# Set API key
export COMET_API_KEY="your-api-key-here"

# Set workspace
export COMET_WORKSPACE="your-workspace"

# Set project
export COMET_PROJECT="your-project"

# Set logging level
export COMET_LOGGING_LEVEL="INFO"
```

## 🔧 Basic Configuration

### 1. **API Key Configuration**
```python
# Method 1: Environment variable
import os
import comet_ml

api_key = os.getenv("COMET_API_KEY")
experiment = comet_ml.Experiment(api_key=api_key)

# Method 2: Direct configuration
import comet_ml

experiment = comet_ml.Experiment(
    api_key="your-api-key-here",
    project_name="my-project"
)

# Method 3: Configuration file
import comet_ml

# Create ~/.cometrc file
experiment = comet_ml.Experiment()  # Reads from config file
```

### 2. **Workspace Configuration**
```python
# Set default workspace
import comet_ml

comet_ml.config.set_workspace("my-workspace")

# Or set per experiment
experiment = comet_ml.Experiment(
    workspace="my-workspace",
    project_name="my-project"
)
```

### 3. **Project Configuration**
```python
# Set default project
import comet_ml

comet_ml.config.set_project("my-project")

# Or set per experiment
experiment = comet_ml.Experiment(
    project_name="my-project"
)
```

## 🔐 Authentication Configuration

### 1. **API Key Management**
```python
# Secure API key handling
import os
from getpass import getpass
import comet_ml

# Get API key securely
api_key = os.getenv("COMET_API_KEY")
if not api_key:
    api_key = getpass("Enter your CometML API key: ")

# Initialize experiment
experiment = comet_ml.Experiment(api_key=api_key)
```

### 2. **OAuth Configuration**
```python
# OAuth authentication (if supported)
import comet_ml

# Configure OAuth
experiment = comet_ml.Experiment(
    oauth_config={
        "client_id": "your-client-id",
        "client_secret": "your-client-secret",
        "redirect_uri": "http://localhost:8080/callback"
    }
)
```

## 📊 Experiment Configuration

### 1. **Auto-logging Configuration**
```python
# Configure auto-logging
experiment = comet_ml.Experiment(
    auto_log_code=True,
    auto_log_system_info=True,
    auto_log_graph=True,
    auto_log_metrics=True,
    auto_log_parameters=True
)
```

### 2. **Custom Logging Configuration**
```python
# Custom logging configuration
import logging
import comet_ml

# Set up logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

# Configure CometML logging
comet_ml.config.set_logging_level("INFO")

experiment = comet_ml.Experiment()
```

### 3. **Experiment Tags and Metadata**
```python
# Configure experiment metadata
experiment = comet_ml.Experiment(
    tags=["production", "v1.0", "important"],
    metadata={
        "author": "data-scientist",
        "team": "ml-team",
        "environment": "production"
    }
)
```

## 🔄 Integration Configuration

### 1. **TensorFlow Integration**
```python
# TensorFlow integration
import comet_ml.tensorflow as comet_tf
import tensorflow as tf

# Configure TensorFlow logging
comet_tf.log_graph = True
comet_tf.log_gradients = True
comet_tf.log_weights = True

# Initialize experiment
experiment = comet_ml.Experiment()

# Use with TensorFlow
with experiment.train():
    model = tf.keras.Sequential([...])
    model.compile(...)
    model.fit(...)
```

### 2. **PyTorch Integration**
```python
# PyTorch integration
import comet_ml.pytorch as comet_torch
import torch

# Configure PyTorch logging
comet_torch.log_graph = True
comet_torch.log_gradients = True
comet_torch.log_weights = True

# Initialize experiment
experiment = comet_ml.Experiment()

# Use with PyTorch
with experiment.train():
    model = torch.nn.Sequential(...)
    optimizer = torch.optim.Adam(model.parameters())
    # Training loop
```

### 3. **Scikit-learn Integration**
```python
# Scikit-learn integration
import comet_ml.sklearn as comet_sklearn
from sklearn.ensemble import RandomForestClassifier

# Configure scikit-learn logging
comet_sklearn.log_model = True
comet_sklearn.log_metrics = True

# Initialize experiment
experiment = comet_ml.Experiment()

# Use with scikit-learn
with experiment.train():
    model = RandomForestClassifier()
    model.fit(X_train, y_train)
```

## ☁️ Cloud Integration Configuration

### 1. **AWS Integration**
```python
# AWS SageMaker integration
import comet_ml

# Configure AWS credentials
experiment = comet_ml.Experiment(
    aws_config={
        "region": "us-west-2",
        "access_key": "your-access-key",
        "secret_key": "your-secret-key"
    }
)

# Deploy to SageMaker
model = experiment.get_model()
model.deploy(
    platform="sagemaker",
    endpoint_name="my-endpoint",
    instance_type="ml.m5.large"
)
```

### 2. **Google Cloud Integration**
```python
# Google Cloud integration
import comet_ml

# Configure GCP credentials
experiment = comet_ml.Experiment(
    gcp_config={
        "project_id": "your-project-id",
        "credentials_path": "/path/to/credentials.json"
    }
)

# Deploy to GCP
model = experiment.get_model()
model.deploy(
    platform="gcp",
    endpoint_name="my-endpoint",
    region="us-central1"
)
```

### 3. **Azure Integration**
```python
# Azure integration
import comet_ml

# Configure Azure credentials
experiment = comet_ml.Experiment(
    azure_config={
        "subscription_id": "your-subscription-id",
        "resource_group": "your-resource-group",
        "workspace_name": "your-workspace"
    }
)

# Deploy to Azure
model = experiment.get_model()
model.deploy(
    platform="azure",
    endpoint_name="my-endpoint",
    region="eastus"
)
```

## 🔧 Advanced Configuration

### 1. **Custom Configuration**
```python
# Custom configuration
import comet_ml

# Set custom configuration
comet_ml.config.set_config({
    "api_key": "your-api-key",
    "workspace": "your-workspace",
    "project": "your-project",
    "auto_log_code": True,
    "auto_log_system_info": True,
    "log_graph": True,
    "log_gradients": True,
    "log_weights": True
})

experiment = comet_ml.Experiment()
```

### 2. **Offline Mode Configuration**
```python
# Offline mode configuration
import comet_ml

# Enable offline mode
experiment = comet_ml.Experiment(
    offline=True,
    offline_directory="./offline_experiments"
)

# Log experiments offline
experiment.log_parameter("test_param", "test_value")
experiment.log_metric("test_metric", 0.95)
experiment.end()

# Upload offline experiments later
comet_ml.upload_offline_experiments(
    offline_directory="./offline_experiments",
    api_key="your-api-key"
)
```

### 3. **Multi-experiment Configuration**
```python
# Multi-experiment configuration
import comet_ml

# Configure multiple experiments
experiments = []
for i in range(5):
    experiment = comet_ml.Experiment(
        project_name=f"experiment-{i}",
        tags=[f"run-{i}", "multi-experiment"]
    )
    experiments.append(experiment)

# Run experiments
for i, experiment in enumerate(experiments):
    with experiment.train():
        # Training code
        experiment.log_metric("accuracy", 0.9 + i * 0.01)
    experiment.end()
```

## 🐛 Common Configuration Issues

### Issue 1: Invalid API Key
```bash
# Error: Invalid API key
# Solution: Check API key
curl -H "Authorization: Bearer $COMET_API_KEY" \
     https://api.comet.ml/api/rest/v2/workspaces
```

### Issue 2: Workspace Not Found
```bash
# Error: Workspace not found
# Solution: Check workspace name
# List available workspaces
curl -H "Authorization: Bearer $COMET_API_KEY" \
     https://api.comet.ml/api/rest/v2/workspaces
```

### Issue 3: Project Access Denied
```bash
# Error: Project access denied
# Solution: Check project permissions
# Verify project access
curl -H "Authorization: Bearer $COMET_API_KEY" \
     https://api.comet.ml/api/rest/v2/projects
```

### Issue 4: Network Connectivity Issues
```bash
# Error: Cannot connect to CometML
# Solution: Check network connectivity
ping api.comet.ml
curl -I https://api.comet.ml/
```

## ✅ Configuration Validation

### Test Configuration
```python
# test_configuration.py
import comet_ml

# Test API connection
api = comet_ml.API()
workspace = api.get_workspace()
print(f"Connected to workspace: {workspace}")

# Test experiment creation
experiment = comet_ml.Experiment(
    project_name="config-test"
)
experiment.log_parameter("test", "value")
experiment.end()
print("Configuration test successful!")
```

### Configuration Checklist
- [ ] API key configured and valid
- [ ] Workspace set and accessible
- [ ] Project configured
- [ ] Auto-logging settings configured
- [ ] Integration settings configured (if needed)
- [ ] Cloud integration configured (if needed)
- [ ] Offline mode configured (if needed)

## 🔧 Configuration Best Practices

1. **🔐 Security**: Use environment variables for API keys
2. **📊 Organization**: Use consistent workspace and project naming
3. **🔄 Automation**: Configure auto-logging for efficiency
4. **☁️ Integration**: Set up cloud integrations for deployment
5. **📝 Documentation**: Document configuration changes
6. **🧪 Testing**: Test configuration in development first

---

*Your CometML configuration is now optimized for your ML workflow! 🎯*
