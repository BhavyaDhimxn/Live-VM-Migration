# WandB — Installation

## 🚀 Installation Methods

WandB can be installed using multiple methods depending on your environment and needs.

## 📦 Method 1: pip (Recommended)

### Basic Installation
```bash
# Install WandB
pip install wandb

# Install with specific extras
pip install wandb[tensorflow]  # TensorFlow integration
pip install wandb[pytorch]      # PyTorch integration
pip install wandb[sklearn]      # Scikit-learn integration
pip install wandb[all]         # All integrations
```

### Virtual Environment (Recommended)
```bash
# Create virtual environment
python -m venv wandb-env
source wandb-env/bin/activate  # On Windows: wandb-env\Scripts\activate

# Install WandB
pip install wandb[all]
```

## 🐳 Method 2: Docker

### WandB Docker Image
```bash
# Pull WandB Docker image
docker pull wandb/client

# Run WandB container
docker run -it --rm \
  -v $(pwd):/workspace \
  -w /workspace \
  -e WANDB_API_KEY=your-api-key \
  wandb/client \
  python your_script.py
```

### Custom Docker Image
```dockerfile
# Dockerfile
FROM python:3.9-slim

# Install WandB
RUN pip install wandb[all]

# Set working directory
WORKDIR /app

# Copy application code
COPY . .

# Set environment variable
ENV WANDB_API_KEY=your-api-key

# Run application
CMD ["python", "main.py"]
```

## ☁️ Method 3: Cloud Installation

### Google Colab
```python
# Install WandB in Google Colab
!pip install wandb

# Import and login
import wandb
wandb.login()
```

### Jupyter Notebook
```bash
# Install WandB for Jupyter
pip install wandb
jupyter nbextension enable --py widgetsnbextension
```

### Kaggle Kernels
```python
# Install WandB in Kaggle
!pip install wandb

# Login
import wandb
wandb.login()
```

## 📋 Prerequisites

### System Requirements
- **Python**: 3.7+ (recommended 3.8+)
- **Memory**: 2GB+ RAM
- **Storage**: 1GB+ free space
- **Network**: Internet access for cloud features

### Required Tools
```bash
# Install Git (for version control)
git --version

# Install Conda (optional, for environment management)
conda --version

# Install Docker (optional, for containerization)
docker --version
```

## ⚙️ Installation Steps

### Step 1: Install WandB
```bash
# Install WandB with all integrations
pip install wandb[all]

# Verify installation
python -c "import wandb; print(wandb.__version__)"
```

### Step 2: Login to WandB
```bash
# Login to WandB
wandb login

# Or set API key as environment variable
export WANDB_API_KEY="your-api-key-here"
```

### Step 3: Initialize WandB
```python
# test_wandb.py
import wandb

# Initialize WandB
wandb.init(
    project="test-project",
    name="test-run"
)

# Test basic functionality
wandb.log({"test_metric": 0.95})
wandb.config.update({"test_param": "test_value"})

# Finish run
wandb.finish()

print("WandB installation successful!")
```

## 🔧 Post-Installation Setup

### 1. Configure API Key
```bash
# Set API key as environment variable
export WANDB_API_KEY="your-api-key-here"

# Or create .netrc file
echo "machine api.wandb.ai login user password your-api-key" > ~/.netrc
chmod 600 ~/.netrc
```

### 2. Configure Default Settings
```bash
# Set default entity (username or team)
wandb login --relogin

# Or set in environment
export WANDB_ENTITY="your-username"
export WANDB_PROJECT="default-project"
```

### 3. Configure WandB Settings
```bash
# Initialize WandB settings
wandb init

# Or create wandb/settings file
mkdir -p wandb
cat > wandb/settings << EOF
[default]
entity = your-username
project = default-project
EOF
```

## ✅ Verification

### Test WandB Installation
```bash
# Check WandB version
python -c "import wandb; print(wandb.__version__)"

# Expected output: 0.x.x
```

### Test Basic Functionality
```python
# test_basic_functionality.py
import wandb
import numpy as np
import matplotlib.pyplot as plt

# Initialize WandB
run = wandb.init(
    project="installation-test",
    name="test-run"
)

# Test config logging
wandb.config.update({
    "test_param": "test_value",
    "learning_rate": 0.01,
    "epochs": 100
})

# Test metric logging
wandb.log({"test_metric": 0.95})
wandb.log({"loss": 0.05})

# Test artifact logging
np.save("test_array.npy", np.random.rand(100, 100))
wandb.log_artifact("test_array.npy", type="dataset")

# Test figure logging
plt.figure()
plt.plot([1, 2, 3, 4], [1, 4, 2, 3])
wandb.log({"test_plot": wandb.Image(plt)})

# Finish run
wandb.finish()

print("All WandB features working correctly!")
```

### Test API Connection
```python
# test_api_connection.py
import wandb

# Test API connection
api = wandb.Api()

# Get user info
user = api.viewer()
print(f"Logged in as: {user.username}")

# List projects
projects = api.projects()
print(f"Available projects: {[p.name for p in projects]}")
```

## 🐛 Common Installation Issues

### Issue 1: Permission Denied
```bash
# Error: Permission denied when installing
# Solution: Use --user flag
pip install --user wandb
```

### Issue 2: Python Version Incompatibility
```bash
# Error: Python version not supported
# Solution: Use Python 3.7+
python --version  # Check version
python3.8 -m pip install wandb  # Use specific version
```

### Issue 3: Missing Dependencies
```bash
# Error: Missing required dependencies
# Solution: Install with extras
pip install wandb[all]
```

### Issue 4: API Key Issues
```bash
# Error: Invalid API key
# Solution: Check API key
echo $WANDB_API_KEY  # Check environment variable
# Or verify key at https://wandb.ai/settings
```

### Issue 5: Network Issues
```bash
# Error: Cannot connect to WandB servers
# Solution: Check network connectivity
ping api.wandb.ai
curl -I https://api.wandb.ai/
```

## 🔍 Troubleshooting Commands

### Check Installation
```bash
# Check WandB installation
pip show wandb

# Check installed packages
pip list | grep wandb

# Check Python path
python -c "import sys; print(sys.path)"
```

### Check Configuration
```bash
# Check environment variables
echo $WANDB_API_KEY
echo $WANDB_ENTITY
echo $WANDB_PROJECT

# Check settings file
cat wandb/settings
```

### Test Network Connectivity
```bash
# Test API connectivity
curl -H "Authorization: Bearer $WANDB_API_KEY" \
     https://api.wandb.ai/api/v1/user

# Test web interface
curl -I https://wandb.ai/
```

## 🚀 Next Steps

After successful installation:

1. **📖 Read**: [Configuration Guide](README_Configuration.md)
2. **🛠️ Setup**: Configure your first experiment
3. **📊 Practice**: Log your first ML experiment
4. **🔄 Learn**: Create your first WandB project

---

*WandB is now ready for your ML experiments! 🎉*