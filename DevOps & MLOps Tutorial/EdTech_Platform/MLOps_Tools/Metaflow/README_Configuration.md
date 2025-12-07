# Metaflow — Configuration

## ⚙️ Configuration Overview

Metaflow configuration involves setting up datastores, metadata services, and cloud execution backends to customize the framework for your specific needs.

## 📁 Configuration Files

### 1. **Metaflow Configuration** (`~/.metaflowconfig`)
```ini
[METAFLOW]
# Data store configuration
DATASTORE_SYSROOT_S3 = s3://my-bucket/metaflow
DATASTORE_SYSROOT_GS = gs://my-bucket/metaflow
DATASTORE_SYSROOT_AZURE = azure://my-container/metaflow

# Metadata service
METADATA_SERVICE_URL = http://metadata-service:8080

# AWS configuration
AWS_DEFAULT_REGION = us-west-2
```

### 2. **Environment Variables**
```bash
# Set data store
export METAFLOW_DATASTORE_SYSROOT_S3="s3://my-bucket/metaflow"

# Set metadata service
export METAFLOW_METADATA_SERVICE_URL="http://metadata-service:8080"

# Set AWS region
export AWS_DEFAULT_REGION="us-west-2"
```

## 🔧 Basic Configuration

### 1. **Local Configuration**
```bash
# Local execution (default)
# No configuration needed
# Metadata stored in .metaflow directory
python my_flow.py run
```

### 2. **S3 Data Store Configuration**
```bash
# Configure S3 data store
export METAFLOW_DATASTORE_SYSROOT_S3="s3://my-bucket/metaflow"

# Configure AWS credentials
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_DEFAULT_REGION="us-west-2"
```

### 3. **Metadata Service Configuration**
```bash
# Configure metadata service
export METAFLOW_METADATA_SERVICE_URL="http://metadata-service:8080"

# Or use local metadata (default)
# No configuration needed
```

## ☁️ Cloud Provider Configuration

### 1. **AWS Configuration**
```bash
# Configure AWS
export AWS_DEFAULT_REGION="us-west-2"
export METAFLOW_DATASTORE_SYSROOT_S3="s3://my-bucket/metaflow"

# Configure AWS Batch
export METAFLOW_BATCH_JOB_QUEUE="my-job-queue"
export METAFLOW_BATCH_JOB_DEFINITION="my-job-definition"
```

### 2. **Google Cloud Configuration**
```bash
# Configure GCP
export METAFLOW_DATASTORE_SYSROOT_GS="gs://my-bucket/metaflow"
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/credentials.json"
```

### 3. **Azure Configuration**
```bash
# Configure Azure
export METAFLOW_DATASTORE_SYSROOT_AZURE="azure://my-container/metaflow"
export AZURE_STORAGE_CONNECTION_STRING="DefaultEndpointsProtocol=https;..."
```

## 🔐 Security Configuration

### 1. **AWS IAM Configuration**
```bash
# Use IAM roles for AWS access
export AWS_PROFILE="my-profile"

# Or use environment variables
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
```

### 2. **Data Store Security**
```bash
# Secure S3 bucket
# Set bucket policy to restrict access
# Use IAM roles for EC2/ECS execution

# Secure GCS bucket
# Set bucket permissions
# Use service accounts
```

## 🔄 Advanced Configuration

### 1. **Batch Execution Configuration**
```python
# Configure batch execution in flow
from metaflow import batch

class MyFlow(FlowSpec):
    @batch(cpu=4, memory=8000, image="my-image:latest")
    @step
    def train(self):
        # Training step with batch resources
        pass
```

### 2. **Kubernetes Configuration**
```python
# Configure Kubernetes execution
from metaflow import kubernetes

class MyFlow(FlowSpec):
    @kubernetes(cpu=4, memory=8000, image="my-image:latest")
    @step
    def train(self):
        # Training step on Kubernetes
        pass
```

### 3. **Card Configuration**
```python
# Configure cards
from metaflow import card

@card(type='html')
@step
def visualize(self):
    # Create visualization card
    current.card.append(create_visualization())
```

## 🐛 Common Configuration Issues

### Issue 1: S3 Access Denied
```bash
# Error: Access denied to S3
# Solution: Check AWS credentials and permissions
aws s3 ls s3://my-bucket/metaflow
```

### Issue 2: Metadata Service Connection Failed
```bash
# Error: Cannot connect to metadata service
# Solution: Check metadata service URL
curl http://metadata-service:8080/health
```

### Issue 3: Batch Job Queue Not Found
```bash
# Error: Batch job queue not found
# Solution: Check AWS Batch configuration
aws batch describe-job-queues --job-queues my-job-queue
```

## ✅ Configuration Validation

### Test Configuration
```bash
# Test local configuration
python my_flow.py show

# Test S3 configuration
python my_flow.py run --datastore s3

# Test batch execution
python my_flow.py run --with batch
```

### Configuration Checklist
- [ ] Data store configured and accessible
- [ ] Metadata service configured (if using remote)
- [ ] Cloud provider credentials configured
- [ ] Batch/Kubernetes configured (if using)
- [ ] Security settings configured
- [ ] Environment variables set

## 🔧 Configuration Best Practices

1. **🔐 Security**: Use IAM roles for cloud access
2. **💾 Data Store**: Use cloud storage for production
3. **📊 Metadata**: Use remote metadata service for teams
4. **🔄 Automation**: Configure for CI/CD integration
5. **📝 Documentation**: Document all configuration changes

---

*Your Metaflow configuration is now optimized for your ML workflows! 🎯*