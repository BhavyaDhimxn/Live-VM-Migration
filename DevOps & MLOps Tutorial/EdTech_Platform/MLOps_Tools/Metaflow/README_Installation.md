# Metaflow — Installation

## 🚀 Installation Methods

Metaflow can be installed using multiple methods depending on your environment and needs.

## 📦 Method 1: pip (Recommended)

### Basic Installation
```bash
# Install Metaflow
pip install metaflow

# Install with specific extras
pip install metaflow[aws]        # AWS integration
pip install metaflow[kubernetes] # Kubernetes integration
pip install metaflow[azure]      # Azure integration
pip install metaflow[all]        # All integrations
```

### Virtual Environment (Recommended)
```bash
# Create virtual environment
python -m venv metaflow-env
source metaflow-env/bin/activate  # On Windows: metaflow-env\Scripts\activate

# Install Metaflow
pip install metaflow[all]
```

## 🐳 Method 2: Docker

### Metaflow Docker Image
```bash
# Pull Metaflow Docker image
docker pull metaflow/metaflow:latest

# Run Metaflow container
docker run -it --rm \
  -v $(pwd):/workspace \
  -w /workspace \
  metaflow/metaflow:latest \
  python my_flow.py run
```

## ☁️ Method 3: Cloud Installation

### AWS Installation
```bash
# Install AWS integration
pip install metaflow[aws]

# Configure AWS credentials
aws configure
# Or set environment variables
export AWS_ACCESS_KEY_ID=your-access-key
export AWS_SECRET_ACCESS_KEY=your-secret-key
export AWS_DEFAULT_REGION=us-west-2
```

### Google Cloud Installation
```bash
# Install GCP integration
pip install metaflow[gcp]

# Configure GCP credentials
gcloud auth application-default login
```

### Azure Installation
```bash
# Install Azure integration
pip install metaflow[azure]

# Configure Azure credentials
az login
```

## 📋 Prerequisites

### System Requirements
- **Python**: 3.7+ (recommended 3.8+)
- **Memory**: 2GB+ RAM
- **Storage**: 1GB+ free space
- **Network**: Internet access for cloud features

### Required Tools
```bash
# Install Python
python --version  # Should be 3.7+

# Install pip
pip --version

# Install Git (for version control)
git --version

# Install Docker (optional, for containerized execution)
docker --version
```

## ⚙️ Installation Steps

### Step 1: Install Metaflow
```bash
# Install Metaflow with all integrations
pip install metaflow[all]

# Verify installation
python -c "import metaflow; print(metaflow.__version__)"
```

### Step 2: Initialize Metaflow
```bash
# Initialize Metaflow (creates .metaflow directory)
python -c "from metaflow import FlowSpec; print('Metaflow initialized')"

# Or run a simple flow
python my_flow.py show
```

### Step 3: Configure Metadata Service (Optional)
```bash
# For local development, metadata is stored locally
# For production, configure remote metadata service

# AWS S3 metadata service
export METAFLOW_DATASTORE_SYSROOT_S3=s3://my-bucket/metaflow
export METAFLOW_METADATA_SERVICE_URL=http://metadata-service:8080
```

## 🔧 Post-Installation Setup

### 1. Configure Local Execution
```bash
# Metaflow works locally by default
# No additional configuration needed for local execution

# Test local execution
python my_flow.py run
```

### 2. Configure Cloud Execution (AWS)
```bash
# Configure AWS for cloud execution
export AWS_DEFAULT_REGION=us-west-2
export METAFLOW_DATASTORE_SYSROOT_S3=s3://my-bucket/metaflow

# Test cloud execution
python my_flow.py run --with batch
```

### 3. Configure Kubernetes Execution
```bash
# Configure Kubernetes
export KUBERNETES_NAMESPACE=metaflow
export KUBERNETES_SERVICE_ACCOUNT=metaflow

# Test Kubernetes execution
python my_flow.py run --with kubernetes
```

## ✅ Verification

### Test Metaflow Installation
```bash
# Check Metaflow version
python -c "import metaflow; print(metaflow.__version__)"

# Expected output: 
# 2.x.x
```

### Test Basic Functionality
```python
# test_metaflow.py
from metaflow import FlowSpec, step

class TestFlow(FlowSpec):
    @step
    def start(self):
        self.message = "Hello Metaflow"
        self.next(self.end)
    
    @step
    def end(self):
        print(self.message)

if __name__ == '__main__':
    TestFlow()

# Run test flow
# python test_metaflow.py run
```

### Test Cloud Execution
```bash
# Test AWS Batch execution
python my_flow.py run --with batch

# Test Kubernetes execution
python my_flow.py run --with kubernetes
```

## 🐛 Common Installation Issues

### Issue 1: Permission Denied
```bash
# Error: Permission denied when installing
# Solution: Use --user flag
pip install --user metaflow[all]
```

### Issue 2: Python Version Incompatibility
```bash
# Error: Python version not supported
# Solution: Use Python 3.7+
python --version  # Check version
python3.8 -m pip install metaflow  # Use specific version
```

### Issue 3: Missing Dependencies
```bash
# Error: Missing required dependencies
# Solution: Install with extras
pip install metaflow[all]
```

### Issue 4: AWS Configuration Issues
```bash
# Error: Cannot connect to AWS
# Solution: Check AWS credentials
aws configure list
aws s3 ls  # Test S3 access
```

### Issue 5: Kubernetes Configuration Issues
```bash
# Error: Cannot connect to Kubernetes
# Solution: Check Kubernetes configuration
kubectl cluster-info
kubectl get nodes
```

## 🔍 Troubleshooting Commands

### Check Installation
```bash
# Check Metaflow installation
pip show metaflow

# Check installed packages
pip list | grep metaflow

# Check Python path
python -c "import sys; print(sys.path)"
```

### Check Configuration
```bash
# Check environment variables
echo $METAFLOW_DATASTORE_SYSROOT_S3
echo $METAFLOW_METADATA_SERVICE_URL
echo $AWS_DEFAULT_REGION

# Check AWS configuration
aws configure list
```

### Test Execution
```bash
# Test local execution
python my_flow.py run

# Test cloud execution
python my_flow.py run --with batch

# View flow
python my_flow.py show
```

## 🚀 Next Steps

After successful installation:

1. **📖 Read**: [Configuration Guide](README_Configuration.md)
2. **🛠️ Setup**: Configure your first flow
3. **📊 Practice**: Create your first workflow
4. **🔄 Learn**: Explore Metaflow features

---

*Metaflow is now ready for your ML workflows! 🎉*