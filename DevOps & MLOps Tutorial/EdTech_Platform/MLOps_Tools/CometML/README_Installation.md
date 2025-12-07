# CometML — Installation

## 🚀 Installation Methods

CometML can be installed using multiple methods depending on your environment and needs.

## 📦 Method 1: pip (Recommended)

### Basic Installation
```bash
# Install CometML
pip install comet-ml

# Install with specific extras
pip install comet-ml[optuna]      # Hyperparameter optimization
pip install comet-ml[tensorflow]  # TensorFlow integration
pip install comet-ml[pytorch]     # PyTorch integration
pip install comet-ml[sklearn]     # Scikit-learn integration
pip install comet-ml[all]         # All integrations
```

### Virtual Environment (Recommended)
```bash
# Create virtual environment
python -m venv comet-env
source comet-env/bin/activate  # On Windows: comet-env\Scripts\activate

# Install CometML
pip install comet-ml[all]
```

## 🐳 Method 2: Docker

### CometML Docker Image
```bash
# Pull CometML Docker image
docker pull cometml/comet-ml

# Run CometML container
docker run -it --rm \
  -v $(pwd):/workspace \
  -w /workspace \
  cometml/comet-ml \
  python your_script.py
```

### Custom Docker Image
```dockerfile
# Dockerfile
FROM python:3.9-slim

# Install CometML
RUN pip install comet-ml[all]

# Set working directory
WORKDIR /app

# Copy application code
COPY . .

# Run application
CMD ["python", "main.py"]
```

## ☁️ Method 3: Cloud Installation

### Google Colab
```python
# Install CometML in Google Colab
!pip install comet-ml

# Import and initialize
import comet_ml
experiment = comet_ml.Experiment()
```

### Jupyter Notebook
```bash
# Install CometML for Jupyter
pip install comet-ml
jupyter nbextension enable --py widgetsnbextension
```

### Kaggle Kernels
```python
# Install CometML in Kaggle
!pip install comet-ml

# Initialize experiment
import comet_ml
experiment = comet_ml.Experiment()
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

### Step 1: Install CometML
```bash
# Install CometML with all integrations
pip install comet-ml[all]

# Verify installation
python -c "import comet_ml; print(comet_ml.__version__)"
```

### Step 2: Get API Key
```bash
# Sign up at https://www.comet.ml/
# Get your API key from the dashboard
# Set environment variable
export COMET_API_KEY="your-api-key-here"
```

### Step 3: Initialize CometML
```python
# test_comet.py
import comet_ml

# Initialize experiment
experiment = comet_ml.Experiment(
    api_key="your-api-key",
    project_name="test-project"
)

# Test basic functionality
experiment.log_parameter("test_param", "test_value")
experiment.log_metric("test_metric", 0.95)

# End experiment
experiment.end()

print("CometML installation successful!")
```

## 🔧 Post-Installation Setup

### 1. Configure API Key
```bash
# Set API key as environment variable
export COMET_API_KEY="your-api-key-here"

# Or create .cometrc file
echo "api_key=your-api-key-here" > ~/.cometrc
```

### 2. Configure Workspace
```python
# Set default workspace
import comet_ml
comet_ml.config.set_workspace("your-workspace")

# Or set in environment
export COMET_WORKSPACE="your-workspace"
```

### 3. Configure Project
```python
# Set default project
import comet_ml
comet_ml.config.set_project("your-project")

# Or set in environment
export COMET_PROJECT="your-project"
```

## ✅ Verification

### Test CometML Installation
```bash
# Check CometML version
python -c "import comet_ml; print(comet_ml.__version__)"

# Expected output: 3.x.x
```

### Test Basic Functionality
```python
# test_basic_functionality.py
import comet_ml
import numpy as np
import matplotlib.pyplot as plt

# Initialize experiment
experiment = comet_ml.Experiment(
    api_key="your-api-key",
    project_name="installation-test"
)

# Test parameter logging
experiment.log_parameter("test_param", "test_value")

# Test metric logging
experiment.log_metric("test_metric", 0.95)

# Test artifact logging
np.save("test_array.npy", np.random.rand(100, 100))
experiment.log_asset("test_array.npy")

# Test figure logging
plt.figure()
plt.plot([1, 2, 3, 4], [1, 4, 2, 3])
experiment.log_figure(plt.gcf(), "test_plot")

# End experiment
experiment.end()

print("All CometML features working correctly!")
```

### Test API Connection
```python
# test_api_connection.py
import comet_ml

# Test API connection
api = comet_ml.API()

# Get workspace info
workspace = api.get_workspace()
print(f"Connected to workspace: {workspace}")

# List projects
projects = api.get_projects()
print(f"Available projects: {[p.name for p in projects]}")
```

## 🐛 Common Installation Issues

### Issue 1: Permission Denied
```bash
# Error: Permission denied when installing
# Solution: Use --user flag
pip install --user comet-ml
```

### Issue 2: Python Version Incompatibility
```bash
# Error: Python version not supported
# Solution: Use Python 3.7+
python --version  # Check version
python3.8 -m pip install comet-ml  # Use specific version
```

### Issue 3: Missing Dependencies
```bash
# Error: Missing required dependencies
# Solution: Install with extras
pip install comet-ml[all]
```

### Issue 4: API Key Issues
```bash
# Error: Invalid API key
# Solution: Check API key
echo $COMET_API_KEY  # Check environment variable
# Or verify key at https://www.comet.ml/
```

### Issue 5: Network Issues
```bash
# Error: Cannot connect to CometML servers
# Solution: Check network connectivity
ping api.comet.ml
curl -I https://api.comet.ml/
```

## 🔍 Troubleshooting Commands

### Check Installation
```bash
# Check CometML installation
pip show comet-ml

# Check installed packages
pip list | grep comet

# Check Python path
python -c "import sys; print(sys.path)"
```

### Check Configuration
```bash
# Check environment variables
echo $COMET_API_KEY
echo $COMET_WORKSPACE
echo $COMET_PROJECT

# Check configuration file
cat ~/.cometrc
```

### Test Network Connectivity
```bash
# Test API connectivity
curl -H "Authorization: Bearer $COMET_API_KEY" \
     https://api.comet.ml/api/rest/v2/workspaces

# Test web interface
curl -I https://www.comet.ml/
```

## 🚀 Next Steps

After successful installation:

1. **📖 Read**: [Configuration Guide](README_Configuration.md)
2. **🛠️ Setup**: Configure your first experiment
3. **📊 Practice**: Log your first ML experiment
4. **🔄 Learn**: Create your first CometML project

---

*CometML is now ready for your ML experiments! 🎉*