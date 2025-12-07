# WandB — Configuration

## ⚙️ Configuration Overview

WandB configuration involves setting up API keys, entities, projects, and various integration settings to customize the platform for your specific needs.

## 📁 Configuration Files

### 1. **WandB Settings** (`wandb/settings`)
```ini
[default]
entity = your-username
project = default-project
base_url = https://api.wandb.ai

[logging]
level = INFO
console = auto

[run]
job_type = training
tags = []
```

### 2. **Environment Variables**
```bash
# Set API key
export WANDB_API_KEY="your-api-key-here"

# Set entity (username or team)
export WANDB_ENTITY="your-username"

# Set default project
export WANDB_PROJECT="default-project"

# Set base URL (for on-premises)
export WANDB_BASE_URL="https://api.wandb.ai"

# Set logging level
export WANDB_LOGGING_LEVEL="INFO"
```

## 🔧 Basic Configuration

### 1. **API Key Configuration**
```python
# Method 1: Environment variable
import os
import wandb

api_key = os.getenv("WANDB_API_KEY")
wandb.login(key=api_key)

# Method 2: Direct login
import wandb

wandb.login(key="your-api-key-here")

# Method 3: Interactive login
import wandb

wandb.login()  # Prompts for API key
```

### 2. **Entity Configuration**
```python
# Set default entity
import wandb

wandb.init(
    entity="your-username",
    project="my-project"
)

# Or set in environment
import os
os.environ["WANDB_ENTITY"] = "your-username"
```

### 3. **Project Configuration**
```python
# Set default project
import wandb

wandb.init(
    project="my-project"
)

# Or set in environment
import os
os.environ["WANDB_PROJECT"] = "my-project"
```

## 🔐 Authentication Configuration

### 1. **API Key Management**
```python
# Secure API key handling
import os
from getpass import getpass
import wandb

def get_api_key():
    """Get API key securely"""
    
    # Try environment variable first
    api_key = os.getenv("WANDB_API_KEY")
    
    if not api_key:
        # Prompt user for API key
        api_key = getpass("Enter your WandB API key: ")
    
    if not api_key:
        raise ValueError("API key is required")
    
    return api_key

# Login with secure API key
wandb.login(key=get_api_key())
```

### 2. **OAuth Configuration**
```python
# OAuth authentication (if supported)
import wandb

# Login with OAuth
wandb.login(relogin=True)
```

## 📊 Experiment Configuration

### 1. **Run Configuration**
```python
# Configure run settings
import wandb

run = wandb.init(
    project="my-project",
    name="experiment-1",
    tags=["production", "v1.0"],
    notes="Initial experiment with baseline model",
    config={
        "learning_rate": 0.01,
        "epochs": 100,
        "batch_size": 32
    }
)
```

### 2. **Auto-logging Configuration**
```python
# Configure auto-logging
import wandb

# TensorFlow auto-logging
wandb.tensorflow.watch(model, log="all")

# PyTorch auto-logging
wandb.watch(model, log="all")

# Keras auto-logging
wandb.keras.log_model(model)
```

### 3. **Custom Logging Configuration**
```python
# Custom logging configuration
import wandb
import logging

# Set up logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

# Configure WandB logging
wandb.init(
    project="my-project",
    settings=wandb.Settings(
        console="auto",
        log_level="info"
    )
)
```

## 🔄 Integration Configuration

### 1. **TensorFlow Integration**
```python
# TensorFlow integration
import wandb
import tensorflow as tf

# Initialize WandB
wandb.init(project="tensorflow-project")

# Configure TensorFlow logging
wandb.tensorflow.watch(
    model,
    log="all",
    log_freq=100
)

# Use with TensorFlow
model = tf.keras.Sequential([...])
model.compile(...)
model.fit(..., callbacks=[wandb.keras.WandbCallback()])
```

### 2. **PyTorch Integration**
```python
# PyTorch integration
import wandb
import torch

# Initialize WandB
wandb.init(project="pytorch-project")

# Configure PyTorch logging
wandb.watch(
    model,
    log="all",
    log_freq=100
)

# Use with PyTorch
model = torch.nn.Sequential(...)
optimizer = torch.optim.Adam(model.parameters())
# Training loop with wandb.log()
```

### 3. **Scikit-learn Integration**
```python
# Scikit-learn integration
import wandb
from sklearn.ensemble import RandomForestClassifier

# Initialize WandB
wandb.init(project="sklearn-project")

# Use with scikit-learn
model = RandomForestClassifier()
wandb.sklearn.plot_learning_curve(model, X, y)
wandb.sklearn.plot_confusion_matrix(y_true, y_pred)
```

## ☁️ Cloud Integration Configuration

### 1. **AWS Integration**
```python
# AWS SageMaker integration
import wandb

# Configure AWS credentials
wandb.init(
    project="aws-project",
    settings=wandb.Settings(
        aws_region="us-west-2",
        aws_access_key="your-access-key",
        aws_secret_key="your-secret-key"
    )
)

# Deploy to SageMaker
wandb.deploy(
    model="model:latest",
    platform="sagemaker",
    endpoint_name="my-endpoint"
)
```

### 2. **Google Cloud Integration**
```python
# Google Cloud integration
import wandb

# Configure GCP credentials
wandb.init(
    project="gcp-project",
    settings=wandb.Settings(
        gcp_project="your-project-id",
        gcp_credentials_path="/path/to/credentials.json"
    )
)

# Deploy to GCP
wandb.deploy(
    model="model:latest",
    platform="gcp",
    endpoint_name="my-endpoint"
)
```

### 3. **Azure Integration**
```python
# Azure integration
import wandb

# Configure Azure credentials
wandb.init(
    project="azure-project",
    settings=wandb.Settings(
        azure_subscription_id="your-subscription-id",
        azure_resource_group="your-resource-group"
    )
)

# Deploy to Azure
wandb.deploy(
    model="model:latest",
    platform="azure",
    endpoint_name="my-endpoint"
)
```

## 🔧 Advanced Configuration

### 1. **Custom Configuration**
```python
# Custom configuration
import wandb

# Set custom configuration
wandb.init(
    project="my-project",
    settings=wandb.Settings(
        console="auto",
        log_level="info",
        _disable_stats=True,
        _disable_meta=True
    )
)
```

### 2. **Offline Mode Configuration**
```python
# Offline mode configuration
import wandb

# Enable offline mode
wandb.init(
    project="my-project",
    mode="offline"
)

# Log experiments offline
wandb.log({"metric": 0.95})
wandb.config.update({"param": "value"})
wandb.finish()

# Sync offline runs later
wandb sync wandb/offline-run-*
```

### 3. **Multi-experiment Configuration**
```python
# Multi-experiment configuration
import wandb

# Configure multiple experiments
experiments = []
for i in range(5):
    run = wandb.init(
        project=f"experiment-{i}",
        name=f"run-{i}",
        tags=[f"run-{i}", "multi-experiment"]
    )
    experiments.append(run)

# Run experiments
for i, run in enumerate(experiments):
    wandb.log({"accuracy": 0.9 + i * 0.01})
    run.finish()
```

## 🐛 Common Configuration Issues

### Issue 1: Invalid API Key
```bash
# Error: Invalid API key
# Solution: Check API key
curl -H "Authorization: Bearer $WANDB_API_KEY" \
     https://api.wandb.ai/api/v1/user
```

### Issue 2: Entity Not Found
```bash
# Error: Entity not found
# Solution: Check entity name
# List available entities
curl -H "Authorization: Bearer $WANDB_API_KEY" \
     https://api.wandb.ai/api/v1/user
```

### Issue 3: Project Access Denied
```bash
# Error: Project access denied
# Solution: Check project permissions
# Verify project access
curl -H "Authorization: Bearer $WANDB_API_KEY" \
     https://api.wandb.ai/api/v1/projects
```

### Issue 4: Network Connectivity Issues
```bash
# Error: Cannot connect to WandB
# Solution: Check network connectivity
ping api.wandb.ai
curl -I https://api.wandb.ai/
```

## ✅ Configuration Validation

### Test Configuration
```python
# test_configuration.py
import wandb

# Test API connection
api = wandb.Api()
user = api.viewer()
print(f"Logged in as: {user.username}")

# Test experiment creation
run = wandb.init(
    project="config-test"
)
wandb.log({"test": "value"})
run.finish()
print("Configuration test successful!")
```

### Configuration Checklist
- [ ] API key configured and valid
- [ ] Entity set and accessible
- [ ] Project configured
- [ ] Auto-logging settings configured
- [ ] Integration settings configured (if needed)
- [ ] Cloud integration configured (if needed)
- [ ] Offline mode configured (if needed)

## 🔧 Configuration Best Practices

1. **🔐 Security**: Use environment variables for API keys
2. **📊 Organization**: Use consistent entity and project naming
3. **🔄 Automation**: Configure auto-logging for efficiency
4. **☁️ Integration**: Set up cloud integrations for deployment
5. **📝 Documentation**: Document configuration changes
6. **🧪 Testing**: Test configuration in development first

---

*Your WandB configuration is now optimized for your ML workflow! 🎯*