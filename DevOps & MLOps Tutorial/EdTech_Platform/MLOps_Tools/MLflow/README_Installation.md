# MLflow — Installation

## 🚀 Installation Methods

MLflow can be installed using multiple methods depending on your needs and environment.

## 📦 Method 1: pip (Recommended)

### Basic Installation
```bash
# Install MLflow
pip install mlflow

# Install with specific extras
pip install mlflow[extras]  # Includes additional dependencies
pip install mlflow[azure]   # Azure integration
pip install mlflow[gcp]     # Google Cloud integration
pip install mlflow[aws]     # AWS integration
```

### Virtual Environment (Recommended)
```bash
# Create virtual environment
python -m venv mlflow-env
source mlflow-env/bin/activate  # On Windows: mlflow-env\Scripts\activate

# Install MLflow
pip install mlflow[extras]
```

## 🐳 Method 2: Docker

### MLflow Server
```bash
# Pull MLflow Docker image
docker pull mlflow/mlflow

# Run MLflow server
docker run -p 5000:5000 mlflow/mlflow
```

### Custom MLflow Image
```dockerfile
# Dockerfile
FROM python:3.9-slim

# Install MLflow
RUN pip install mlflow[extras]

# Set working directory
WORKDIR /mlflow

# Expose port
EXPOSE 5000

# Run MLflow server
CMD ["mlflow", "server", "--host", "0.0.0.0", "--port", "5000"]
```

## ☁️ Method 3: Cloud Installation

### AWS SageMaker
```bash
# Install SageMaker integration
pip install mlflow[sagemaker]

# Deploy to SageMaker
mlflow sagemaker deploy -m "models:/my-model/1" -n "my-endpoint"
```

### Google Cloud Platform
```bash
# Install GCP integration
pip install mlflow[gcp]

# Deploy to GCP
mlflow models build-docker -m "models:/my-model/1" -n "my-model"
```

### Microsoft Azure
```bash
# Install Azure integration
pip install mlflow[azure]

# Deploy to Azure
mlflow azureml deploy -m "models:/my-model/1" -n "my-model"
```

## 📋 Prerequisites

### System Requirements
- **Python**: 3.7+ (recommended 3.8+)
- **Memory**: 2GB+ RAM
- **Storage**: 1GB+ free space
- **Network**: Internet access for package installation

### Required Tools
```bash
# Install Git (for MLflow Projects)
git --version

# Install Conda (optional, for conda environments)
conda --version

# Install Docker (optional, for containerization)
docker --version
```

## ⚙️ Installation Steps

### Step 1: Install MLflow
```bash
# Install MLflow with all extras
pip install mlflow[extras]

# Verify installation
mlflow --version
```

### Step 2: Start MLflow Server
```bash
# Start local MLflow server
mlflow server --host 0.0.0.0 --port 5000

# Access UI at http://localhost:5000
```

### Step 3: Configure Backend Store
```bash
# Start with SQLite backend
mlflow server --backend-store-uri sqlite:///mlflow.db

# Start with PostgreSQL backend
mlflow server --backend-store-uri postgresql://user:password@localhost/mlflow

# Start with MySQL backend
mlflow server --backend-store-uri mysql://user:password@localhost/mlflow
```

## 🔧 Post-Installation Setup

### 1. Configure MLflow Tracking
```python
# Set tracking URI
import mlflow
mlflow.set_tracking_uri("http://localhost:5000")

# Create experiment
mlflow.create_experiment("my-experiment")

# Set experiment
mlflow.set_experiment("my-experiment")
```

### 2. Configure Artifact Storage
```bash
# Start with local artifact storage
mlflow server --default-artifact-root ./mlruns

# Start with S3 artifact storage
mlflow server --default-artifact-root s3://my-bucket/mlflow-artifacts

# Start with GCS artifact storage
mlflow server --default-artifact-root gs://my-bucket/mlflow-artifacts
```

### 3. Configure Model Registry
```bash
# Start with model registry enabled
mlflow server --backend-store-uri sqlite:///mlflow.db --default-artifact-root ./mlruns
```

## ✅ Verification

### Test MLflow Installation
```bash
# Check MLflow version
mlflow --version

# Expected output: mlflow, version 2.x.x
```

### Test Basic Functionality
```python
# test_mlflow.py
import mlflow
import mlflow.sklearn
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split

# Create sample data
X, y = make_classification(n_samples=1000, n_features=4, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Train model
model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)

# Test MLflow tracking
with mlflow.start_run():
    # Log parameters
    mlflow.log_param("n_estimators", 100)
    mlflow.log_param("random_state", 42)
    
    # Log metrics
    accuracy = model.score(X_test, y_test)
    mlflow.log_metric("accuracy", accuracy)
    
    # Log model
    mlflow.sklearn.log_model(model, "model")
    
    print(f"Model accuracy: {accuracy}")
    print("MLflow tracking test successful!")
```

### Test MLflow Server
```bash
# Check server status
curl http://localhost:5000/health

# Expected response: {"status": "ok"}
```

## 🐛 Common Installation Issues

### Issue 1: Permission Denied
```bash
# Error: Permission denied when installing
# Solution: Use --user flag
pip install --user mlflow
```

### Issue 2: Python Version Incompatibility
```bash
# Error: Python version not supported
# Solution: Use Python 3.7+
python --version  # Check version
python3.8 -m pip install mlflow  # Use specific version
```

### Issue 3: Missing Dependencies
```bash
# Error: Missing required dependencies
# Solution: Install with extras
pip install mlflow[extras]
```

### Issue 4: Database Connection Issues
```bash
# Error: Cannot connect to database
# Solution: Check database configuration
# For PostgreSQL:
psql -h localhost -U username -d mlflow

# For MySQL:
mysql -h localhost -u username -p mlflow
```

## 🔍 Troubleshooting Commands

### Check MLflow Status
```bash
# Check MLflow version and installation
mlflow --version
pip show mlflow

# Check server status
curl http://localhost:5000/health

# Check database connection
mlflow server --backend-store-uri sqlite:///test.db --dry-run
```

### Monitor MLflow Server
```bash
# Check server logs
mlflow server --host 0.0.0.0 --port 5000 --verbose

# Check server processes
ps aux | grep mlflow

# Check server ports
netstat -tlnp | grep 5000
```

## 🚀 Next Steps

After successful installation:

1. **📖 Read**: [Configuration Guide](README_Configuration.md)
2. **🛠️ Setup**: Configure your first experiment
3. **📊 Practice**: Log your first ML experiment
4. **🔄 Learn**: Create your first MLflow project

---

*MLflow is now ready for your ML experiments! 🎉*
