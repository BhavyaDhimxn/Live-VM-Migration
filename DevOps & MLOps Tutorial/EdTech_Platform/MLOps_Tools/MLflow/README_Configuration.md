# MLflow — Configuration

## ⚙️ Configuration Overview

MLflow configuration involves setting up tracking servers, backend stores, artifact storage, and model registry to suit your specific needs.

## 📁 Configuration Files

### 1. **MLflow Configuration** (`mlflow.conf`)
```ini
[mlflow]
# Tracking server configuration
tracking_uri = http://localhost:5000

# Backend store configuration
backend_store_uri = sqlite:///mlflow.db

# Artifact store configuration
default_artifact_root = ./mlruns

# Model registry configuration
model_registry_uri = sqlite:///mlflow.db
```

### 2. **Environment Variables**
```bash
# Set tracking URI
export MLFLOW_TRACKING_URI="http://localhost:5000"

# Set backend store
export MLFLOW_BACKEND_STORE_URI="sqlite:///mlflow.db"

# Set artifact root
export MLFLOW_DEFAULT_ARTIFACT_ROOT="./mlruns"

# Set model registry
export MLFLOW_MODEL_REGISTRY_URI="sqlite:///mlflow.db"
```

## 🔧 Backend Store Configuration

### 1. **SQLite Configuration**
```bash
# Start MLflow with SQLite backend
mlflow server --backend-store-uri sqlite:///mlflow.db

# Or with custom database file
mlflow server --backend-store-uri sqlite:///path/to/mlflow.db
```

### 2. **PostgreSQL Configuration**
```bash
# Install PostgreSQL dependencies
pip install psycopg2-binary

# Start MLflow with PostgreSQL backend
mlflow server --backend-store-uri postgresql://user:password@localhost:5432/mlflow

# Or with connection string
mlflow server --backend-store-uri "postgresql://user:password@localhost:5432/mlflow?sslmode=require"
```

### 3. **MySQL Configuration**
```bash
# Install MySQL dependencies
pip install pymysql

# Start MLflow with MySQL backend
mlflow server --backend-store-uri mysql+pymysql://user:password@localhost:3306/mlflow

# Or with connection string
mlflow server --backend-store-uri "mysql+pymysql://user:password@localhost:3306/mlflow?charset=utf8mb4"
```

## ☁️ Artifact Storage Configuration

### 1. **Local Storage**
```bash
# Start with local artifact storage
mlflow server --default-artifact-root ./mlruns

# Or with custom path
mlflow server --default-artifact-root /path/to/artifacts
```

### 2. **AWS S3 Configuration**
```bash
# Install S3 dependencies
pip install boto3

# Start with S3 artifact storage
mlflow server --default-artifact-root s3://my-bucket/mlflow-artifacts

# Configure AWS credentials
export AWS_ACCESS_KEY_ID=your_access_key
export AWS_SECRET_ACCESS_KEY=your_secret_key
export AWS_DEFAULT_REGION=us-west-2
```

### 3. **Google Cloud Storage Configuration**
```bash
# Install GCS dependencies
pip install google-cloud-storage

# Start with GCS artifact storage
mlflow server --default-artifact-root gs://my-bucket/mlflow-artifacts

# Configure GCS credentials
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/credentials.json
```

### 4. **Azure Blob Storage Configuration**
```bash
# Install Azure dependencies
pip install azure-storage-blob

# Start with Azure artifact storage
mlflow server --default-artifact-root wasbs://container@account.blob.core.windows.net/mlflow-artifacts

# Configure Azure credentials
export AZURE_STORAGE_CONNECTION_STRING="DefaultEndpointsProtocol=https;AccountName=account;AccountKey=key"
```

## 🔐 Authentication Configuration

### 1. **Basic Authentication**
```bash
# Start MLflow with basic auth
mlflow server --app-name basic-auth --backend-store-uri sqlite:///mlflow.db

# Configure authentication in code
import mlflow
mlflow.set_tracking_uri("http://username:password@localhost:5000")
```

### 2. **OAuth Configuration**
```bash
# Start MLflow with OAuth
mlflow server --app-name oauth --backend-store-uri sqlite:///mlflow.db

# Configure OAuth in code
import mlflow
mlflow.set_tracking_uri("http://localhost:5000")
mlflow.set_experiment("my-experiment")
```

## 📊 Model Registry Configuration

### 1. **Enable Model Registry**
```bash
# Start MLflow with model registry
mlflow server --backend-store-uri sqlite:///mlflow.db --default-artifact-root ./mlruns
```

### 2. **Configure Model Registry**
```python
# Configure model registry in code
import mlflow
from mlflow.tracking import MlflowClient

# Set tracking URI
mlflow.set_tracking_uri("http://localhost:5000")

# Create client
client = MlflowClient()

# Register model
model_version = client.create_registered_model("my-model")
```

## 🔄 Advanced Configuration

### 1. **Custom Server Configuration**
```bash
# Start MLflow with custom configuration
mlflow server \
  --host 0.0.0.0 \
  --port 5000 \
  --backend-store-uri postgresql://user:password@localhost:5432/mlflow \
  --default-artifact-root s3://my-bucket/mlflow-artifacts \
  --workers 4 \
  --gunicorn-opts "--timeout 120"
```

### 2. **Docker Configuration**
```dockerfile
# Dockerfile for MLflow server
FROM python:3.9-slim

# Install MLflow
RUN pip install mlflow[extras] psycopg2-binary boto3

# Set environment variables
ENV MLFLOW_BACKEND_STORE_URI=postgresql://user:password@db:5432/mlflow
ENV MLFLOW_DEFAULT_ARTIFACT_ROOT=s3://my-bucket/mlflow-artifacts

# Expose port
EXPOSE 5000

# Run MLflow server
CMD ["mlflow", "server", "--host", "0.0.0.0", "--port", "5000"]
```

### 3. **Kubernetes Configuration**
```yaml
# mlflow-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mlflow-server
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mlflow-server
  template:
    metadata:
      labels:
        app: mlflow-server
    spec:
      containers:
      - name: mlflow
        image: mlflow/mlflow:latest
        ports:
        - containerPort: 5000
        env:
        - name: MLFLOW_BACKEND_STORE_URI
          value: "postgresql://user:password@postgres:5432/mlflow"
        - name: MLFLOW_DEFAULT_ARTIFACT_ROOT
          value: "s3://my-bucket/mlflow-artifacts"
        resources:
          requests:
            cpu: "1"
            memory: "2Gi"
          limits:
            cpu: "2"
            memory: "4Gi"
```

## 🔧 Client Configuration

### 1. **Python Client Configuration**
```python
# mlflow_client_config.py
import mlflow
from mlflow.tracking import MlflowClient

# Set tracking URI
mlflow.set_tracking_uri("http://localhost:5000")

# Create client
client = MlflowClient()

# Set experiment
mlflow.set_experiment("my-experiment")

# Configure logging
import logging
logging.basicConfig(level=logging.INFO)
mlflow.set_log_level("INFO")
```

### 2. **R Client Configuration**
```r
# mlflow_r_config.R
library(mlflow)

# Set tracking URI
mlflow_set_tracking_uri("http://localhost:5000")

# Set experiment
mlflow_set_experiment("my-experiment")

# Configure logging
mlflow_set_log_level("INFO")
```

### 3. **Java Client Configuration**
```java
// MLflowClientConfig.java
import org.mlflow.tracking.MlflowClient;
import org.mlflow.tracking.MlflowContext;

// Create client
MlflowClient client = new MlflowClient("http://localhost:5000");

// Set experiment
MlflowContext context = new MlflowContext();
context.setExperimentId("my-experiment");
```

## 🐛 Common Configuration Issues

### Issue 1: Database Connection Failed
```bash
# Error: Cannot connect to database
# Solution: Check database configuration
# For PostgreSQL:
psql -h localhost -U username -d mlflow

# For MySQL:
mysql -h localhost -u username -p mlflow
```

### Issue 2: Artifact Storage Access Denied
```bash
# Error: Access denied to artifact storage
# Solution: Check credentials and permissions
# For S3:
aws s3 ls s3://my-bucket/mlflow-artifacts

# For GCS:
gsutil ls gs://my-bucket/mlflow-artifacts
```

### Issue 3: Model Registry Not Found
```bash
# Error: Model registry not found
# Solution: Enable model registry
mlflow server --backend-store-uri sqlite:///mlflow.db --default-artifact-root ./mlruns
```

### Issue 4: Authentication Failed
```bash
# Error: Authentication failed
# Solution: Check credentials
curl -u username:password http://localhost:5000/health
```

## ✅ Configuration Validation

### Test Configuration
```bash
# Test tracking server
curl http://localhost:5000/health

# Test database connection
mlflow server --backend-store-uri sqlite:///test.db --dry-run

# Test artifact storage
mlflow server --default-artifact-root ./test-artifacts --dry-run
```

### Configuration Checklist
- [ ] Tracking server accessible
- [ ] Backend store configured and accessible
- [ ] Artifact storage configured and accessible
- [ ] Model registry enabled (if needed)
- [ ] Authentication configured (if needed)
- [ ] Client configuration working
- [ ] Environment variables set

## 🔧 Configuration Best Practices

1. **🔐 Security**: Use secure authentication and encrypted connections
2. **💾 Backup**: Regular backups of backend store and artifacts
3. **📊 Monitoring**: Monitor server performance and resource usage
4. **🔄 Scaling**: Configure for horizontal scaling if needed
5. **📝 Documentation**: Document all configuration changes
6. **🧪 Testing**: Test configuration in staging environment first

---

*Your MLflow configuration is now optimized for your ML workflow! 🎯*
