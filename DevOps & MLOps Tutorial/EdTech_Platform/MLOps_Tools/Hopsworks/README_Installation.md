# Hopsworks — Installation

## 🚀 Installation Methods

Hopsworks can be installed using multiple methods depending on your environment and needs.

## 📦 Method 1: pip (Python Client)

### Basic Installation
```bash
# Install Hopsworks Python client
pip install hopsworks

# Install with specific extras
pip install hopsworks[spark]      # Spark integration
pip install hopsworks[tensorflow] # TensorFlow integration
pip install hopsworks[pytorch]    # PyTorch integration
```

### Virtual Environment (Recommended)
```bash
# Create virtual environment
python -m venv hopsworks-env
source hopsworks-env/bin/activate  # On Windows: hopsworks-env\Scripts\activate

# Install Hopsworks
pip install hopsworks[all]
```

## 🐳 Method 2: Docker

### Hopsworks Docker Image
```bash
# Pull Hopsworks Docker image
docker pull logicalclocks/hopsworks:latest

# Run Hopsworks container
docker run -it --rm \
  -v $(pwd):/workspace \
  -w /workspace \
  logicalclocks/hopsworks:latest \
  python your_script.py
```

### Docker Compose (Full Stack)
```yaml
# docker-compose.yml
version: '3.8'
services:
  hopsworks:
    image: logicalclocks/hopsworks:latest
    ports:
      - "8181:8181"
    environment:
      - HOPSWORKS_API_KEY=your-api-key
    volumes:
      - ./workspace:/workspace
```

## ☁️ Method 3: Cloud Installation

### AWS Installation
```bash
# Install AWS providers
pip install hopsworks[aws]

# Configure AWS credentials
aws configure
# Or set environment variables
export AWS_ACCESS_KEY_ID=your-access-key
export AWS_SECRET_ACCESS_KEY=your-secret-key
```

### Google Cloud Installation
```bash
# Install GCP providers
pip install hopsworks[gcp]

# Configure GCP credentials
gcloud auth application-default login
```

### Azure Installation
```bash
# Install Azure providers
pip install hopsworks[azure]

# Configure Azure credentials
az login
```

## 📋 Prerequisites

### System Requirements
- **Python**: 3.7+ (recommended 3.8+)
- **Memory**: 4GB+ RAM (8GB+ recommended)
- **Storage**: 10GB+ free space
- **Network**: Internet access for cloud features

### Required Tools
```bash
# Install Python
python --version  # Should be 3.7+

# Install pip
pip --version

# Install Git (for version control)
git --version

# Install Docker (optional, for containerized deployment)
docker --version
```

## ⚙️ Installation Steps

### Step 1: Install Hopsworks Client
```bash
# Install Hopsworks Python client
pip install hopsworks[all]

# Verify installation
python -c "import hopsworks; print(hopsworks.__version__)"
```

### Step 2: Get API Key
```bash
# Sign up at https://www.hopsworks.ai/
# Get your API key from the dashboard
# Set environment variable
export HOPSWORKS_API_KEY="your-api-key-here"
```

### Step 3: Initialize Connection
```python
# test_hopsworks.py
import hopsworks

# Connect to Hopsworks
project = hopsworks.login(
    api_key_value="your-api-key",
    project="my_project"
)

print(f"Connected to project: {project.name}")
```

## 🔧 Post-Installation Setup

### 1. Configure API Key
```bash
# Set API key as environment variable
export HOPSWORKS_API_KEY="your-api-key-here"

# Or create .hopsworksrc file
echo "api_key=your-api-key-here" > ~/.hopsworksrc
```

### 2. Configure Project
```python
# Set default project
import hopsworks

project = hopsworks.login(
    api_key_value=os.getenv("HOPSWORKS_API_KEY"),
    project="my_project"
)
```

### 3. Set Up Feature Store
```python
# Initialize feature store
import hsfs

connection = hsfs.connection()
fs = connection.get_feature_store("my_feature_store")

print(f"Feature store: {fs.name}")
```

## ✅ Verification

### Test Hopsworks Installation
```bash
# Check Hopsworks version
python -c "import hopsworks; print(hopsworks.__version__)"

# Expected output: 
# 3.x.x
```

### Test Basic Functionality
```python
# test_basic_functionality.py
import hopsworks
import hsfs

# Connect to Hopsworks
project = hopsworks.login(
    api_key_value="your-api-key",
    project="my_project"
)

# Test feature store
connection = hsfs.connection()
fs = connection.get_feature_store("my_feature_store")

print(f"Feature store initialized: {fs.name}")

# List feature groups
feature_groups = fs.get_feature_groups()
print(f"Feature groups: {[fg.name for fg in feature_groups]}")
```

### Test Model Registry
```python
# test_model_registry.py
import hsml

# Connect to model registry
mr = hsml.connection().get_model_registry()

# List models
models = mr.get_models()
print(f"Models: {[m.name for m in models]}")
```

## 🐛 Common Installation Issues

### Issue 1: Permission Denied
```bash
# Error: Permission denied when installing
# Solution: Use --user flag
pip install --user hopsworks[all]
```

### Issue 2: Python Version Incompatibility
```bash
# Error: Python version not supported
# Solution: Use Python 3.7+
python --version  # Check version
python3.8 -m pip install hopsworks  # Use specific version
```

### Issue 3: Missing Dependencies
```bash
# Error: Missing required dependencies
# Solution: Install with extras
pip install hopsworks[all]
```

### Issue 4: API Key Issues
```bash
# Error: Invalid API key
# Solution: Check API key
echo $HOPSWORKS_API_KEY  # Check environment variable
# Or verify key at https://www.hopsworks.ai/
```

### Issue 5: Connection Issues
```bash
# Error: Cannot connect to Hopsworks
# Solution: Check network connectivity
ping app.hopsworks.ai
curl -I https://app.hopsworks.ai/
```

## 🔍 Troubleshooting Commands

### Check Installation
```bash
# Check Hopsworks installation
pip show hopsworks

# Check installed packages
pip list | grep hopsworks

# Check Python path
python -c "import sys; print(sys.path)"
```

### Check Configuration
```bash
# Check environment variables
echo $HOPSWORKS_API_KEY
echo $HOPSWORKS_PROJECT

# Check configuration file
cat ~/.hopsworksrc
```

### Test Network Connectivity
```bash
# Test API connectivity
curl -H "Authorization: Bearer $HOPSWORKS_API_KEY" \
     https://app.hopsworks.ai/api/projects

# Test web interface
curl -I https://app.hopsworks.ai/
```

## 🚀 Next Steps

After successful installation:

1. **📖 Read**: [Configuration Guide](README_Configuration.md)
2. **🛠️ Setup**: Configure your first feature store
3. **📊 Practice**: Create your first feature group
4. **🔄 Learn**: Build your first ML pipeline

---

*Hopsworks is now ready for your MLOps operations! 🎉*