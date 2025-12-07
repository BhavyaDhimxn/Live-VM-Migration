# DataRobot — Installation

## 🚀 Installation Methods

DataRobot can be accessed through its cloud platform or installed on-premises. The Python client is the primary way to interact with DataRobot programmatically.

## 📦 Method 1: Python Client (Recommended)

### Basic Installation
```bash
# Install DataRobot Python client
pip install datarobot

# Install with specific extras
pip install datarobot[all]  # All dependencies
```

### Virtual Environment (Recommended)
```bash
# Create virtual environment
python -m venv datarobot-env
source datarobot-env/bin/activate  # On Windows: datarobot-env\Scripts\activate

# Install DataRobot
pip install datarobot
```

## ☁️ Method 2: Cloud Platform Access

### Sign Up for DataRobot Cloud
```bash
# 1. Sign up at https://www.datarobot.com/
# 2. Get API token from dashboard
# 3. Set environment variable
export DATAROBOT_API_TOKEN="your-api-token"
export DATAROBOT_ENDPOINT="https://app.datarobot.com/api/v2"
```

### Access Web UI
```bash
# Access DataRobot web interface
# URL: https://app.datarobot.com
# Login with your credentials
```

## 🏠 Method 3: On-Premises Installation

### Docker Installation
```bash
# Pull DataRobot Docker image
docker pull datarobot/datarobot:latest

# Run DataRobot container
docker run -d \
  -p 8080:8080 \
  -v datarobot-data:/data \
  datarobot/datarobot:latest
```

### Kubernetes Installation
```yaml
# datarobot-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: datarobot
spec:
  replicas: 1
  selector:
    matchLabels:
      app: datarobot
  template:
    metadata:
      labels:
        app: datarobot
    spec:
      containers:
      - name: datarobot
        image: datarobot/datarobot:latest
        ports:
        - containerPort: 8080
```

## 📋 Prerequisites

### System Requirements
- **Python**: 3.7+ (recommended 3.8+)
- **Memory**: 4GB+ RAM (8GB+ recommended)
- **Storage**: 10GB+ free space
- **Network**: Internet access for cloud platform

### Required Tools
```bash
# Install Python
python --version  # Should be 3.7+

# Install pip
pip --version

# Install Git (for version control)
git --version
```

## ⚙️ Installation Steps

### Step 1: Install Python Client
```bash
# Install DataRobot Python client
pip install datarobot

# Verify installation
python -c "import datarobot; print(datarobot.__version__)"
```

### Step 2: Get API Token
```bash
# Sign up at https://www.datarobot.com/
# Get API token from dashboard:
# Settings > API Tokens > Create Token

# Set environment variable
export DATAROBOT_API_TOKEN="your-api-token"
export DATAROBOT_ENDPOINT="https://app.datarobot.com/api/v2"
```

### Step 3: Test Connection
```python
# test_connection.py
import datarobot as dr

# Connect to DataRobot
client = dr.Client(
    token='your-api-token',
    endpoint='https://app.datarobot.com/api/v2'
)

# Test connection
projects = dr.Project.list()
print(f"Connected! Found {len(projects)} projects")
```

## 🔧 Post-Installation Setup

### 1. Configure API Token
```bash
# Set API token as environment variable
export DATAROBOT_API_TOKEN="your-api-token"

# Or create .datarobotrc file
echo "api_token=your-api-token" > ~/.datarobotrc
echo "endpoint=https://app.datarobot.com/api/v2" >> ~/.datarobotrc
```

### 2. Configure Default Project
```python
# Set default project
import datarobot as dr

client = dr.Client(
    token=os.getenv("DATAROBOT_API_TOKEN"),
    endpoint=os.getenv("DATAROBOT_ENDPOINT")
)

# List projects
projects = dr.Project.list()
print(f"Available projects: {[p.project_name for p in projects]}")
```

### 3. Set Up Workspace
```python
# Create workspace structure
import os

workspace_dir = "datarobot_workspace"
os.makedirs(workspace_dir, exist_ok=True)
os.makedirs(f"{workspace_dir}/data", exist_ok=True)
os.makedirs(f"{workspace_dir}/models", exist_ok=True)
os.makedirs(f"{workspace_dir}/notebooks", exist_ok=True)
```

## ✅ Verification

### Test DataRobot Installation
```bash
# Check DataRobot version
python -c "import datarobot; print(datarobot.__version__)"

# Expected output: 
# 3.x.x
```

### Test Basic Functionality
```python
# test_basic_functionality.py
import datarobot as dr

# Connect to DataRobot
client = dr.Client(
    token='your-api-token',
    endpoint='https://app.datarobot.com/api/v2'
)

# Create test project
project = dr.Project.create(
    project_name='test-project',
    sourcedata='data/test.csv'
)

print(f"Project created: {project.id}")

# List projects
projects = dr.Project.list()
print(f"Total projects: {len(projects)}")
```

### Test API Connection
```python
# test_api_connection.py
import datarobot as dr

# Test API connection
try:
    client = dr.Client(
        token='your-api-token',
        endpoint='https://app.datarobot.com/api/v2'
    )
    
    # Get user info
    user = dr.User.get()
    print(f"Connected as: {user.username}")
    
except Exception as e:
    print(f"Connection failed: {e}")
```

## 🐛 Common Installation Issues

### Issue 1: Permission Denied
```bash
# Error: Permission denied when installing
# Solution: Use --user flag
pip install --user datarobot
```

### Issue 2: Python Version Incompatibility
```bash
# Error: Python version not supported
# Solution: Use Python 3.7+
python --version  # Check version
python3.8 -m pip install datarobot  # Use specific version
```

### Issue 3: API Token Issues
```bash
# Error: Invalid API token
# Solution: Check API token
echo $DATAROBOT_API_TOKEN  # Check environment variable
# Or verify token at https://app.datarobot.com/
```

### Issue 4: Connection Issues
```bash
# Error: Cannot connect to DataRobot
# Solution: Check network connectivity
ping app.datarobot.com
curl -I https://app.datarobot.com/
```

### Issue 5: Missing Dependencies
```bash
# Error: Missing required dependencies
# Solution: Install with all extras
pip install datarobot[all]
```

## 🔍 Troubleshooting Commands

### Check Installation
```bash
# Check DataRobot installation
pip show datarobot

# Check installed packages
pip list | grep datarobot

# Check Python path
python -c "import sys; print(sys.path)"
```

### Check Configuration
```bash
# Check environment variables
echo $DATAROBOT_API_TOKEN
echo $DATAROBOT_ENDPOINT

# Check configuration file
cat ~/.datarobotrc
```

### Test Network Connectivity
```bash
# Test API connectivity
curl -H "Authorization: Bearer $DATAROBOT_API_TOKEN" \
     https://app.datarobot.com/api/v2/projects

# Test web interface
curl -I https://app.datarobot.com/
```

## 🚀 Next Steps

After successful installation:

1. **📖 Read**: [Configuration Guide](README_Configuration.md)
2. **🛠️ Setup**: Configure your first project
3. **📊 Practice**: Create your first AutoML project
4. **🔄 Learn**: Explore DataRobot features

---

*DataRobot is now ready for your automated ML operations! 🎉*