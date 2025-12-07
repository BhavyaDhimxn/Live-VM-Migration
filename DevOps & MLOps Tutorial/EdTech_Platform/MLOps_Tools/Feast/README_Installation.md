# Feast — Installation

## 🚀 Installation Methods

Feast can be installed using multiple methods depending on your environment and needs.

## 📦 Method 1: pip (Recommended)

### Basic Installation
```bash
# Install Feast
pip install feast

# Install with specific providers
pip install feast[redis]           # Redis online store
pip install feast[postgres]         # PostgreSQL online store
pip install feast[gcp]              # Google Cloud providers
pip install feast[aws]               # AWS providers
pip install feast[azure]             # Azure providers
pip install feast[all]              # All providers
```

### Virtual Environment (Recommended)
```bash
# Create virtual environment
python -m venv feast-env
source feast-env/bin/activate  # On Windows: feast-env\Scripts\activate

# Install Feast
pip install feast[all]
```

## 🐳 Method 2: Docker

### Feast Docker Image
```bash
# Pull Feast Docker image
docker pull feastdev/feast-sdk:latest

# Run Feast container
docker run -it --rm \
  -v $(pwd):/workspace \
  -w /workspace \
  feastdev/feast-sdk:latest \
  feast --help
```

### Docker Compose (Full Stack)
```yaml
# docker-compose.yml
version: '3.8'
services:
  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
  
  postgres:
    image: postgres:13
    environment:
      POSTGRES_DB: feast
      POSTGRES_USER: feast
      POSTGRES_PASSWORD: feast
    ports:
      - "5432:5432"
  
  feast:
    image: feastdev/feast-sdk:latest
    volumes:
      - ./feast_repo:/workspace
    working_dir: /workspace
    command: feast serve
```

## ☁️ Method 3: Cloud Installation

### AWS Installation
```bash
# Install AWS providers
pip install feast[aws]

# Configure AWS credentials
aws configure
# Or set environment variables
export AWS_ACCESS_KEY_ID=your-access-key
export AWS_SECRET_ACCESS_KEY=your-secret-key
```

### Google Cloud Installation
```bash
# Install GCP providers
pip install feast[gcp]

# Configure GCP credentials
gcloud auth application-default login
# Or set environment variable
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/credentials.json
```

### Azure Installation
```bash
# Install Azure providers
pip install feast[azure]

# Configure Azure credentials
az login
# Or set environment variables
export AZURE_STORAGE_CONNECTION_STRING="DefaultEndpointsProtocol=https;..."
```

## 📋 Prerequisites

### System Requirements
- **Python**: 3.7+ (recommended 3.8+)
- **Memory**: 2GB+ RAM
- **Storage**: 1GB+ free space
- **Database**: PostgreSQL, MySQL, or SQLite (for registry)

### Required Tools
```bash
# Install Python
python --version  # Should be 3.7+

# Install pip
pip --version

# Install Git (for feature repository version control)
git --version

# Install Docker (optional, for containerized deployment)
docker --version
```

## ⚙️ Installation Steps

### Step 1: Install Feast
```bash
# Install Feast with all providers
pip install feast[all]

# Verify installation
feast --version
```

### Step 2: Initialize Feature Store
```bash
# Create feature store repository
feast init my_feature_store
cd my_feature_store

# This creates:
# - feature_store.yaml (configuration)
# - data/ (data directory)
# - example_repo.py (example feature definitions)
```

### Step 3: Configure Feature Store
```bash
# Edit feature_store.yaml
nano feature_store.yaml

# Basic configuration:
# project: my_feature_store
# registry: data/registry.db
# provider: local
```

### Step 4: Test Installation
```bash
# Test Feast CLI
feast --help

# Test feature store initialization
feast apply

# Test feature retrieval
feast materialize-incremental $(date +%Y-%m-%d)
```

## 🔧 Post-Installation Setup

### 1. Configure Online Store
```yaml
# feature_store.yaml
project: my_feature_store
registry: data/registry.db
provider: local
online_store:
    type: redis
    connection_string: "localhost:6379"
```

### 2. Configure Offline Store
```yaml
# feature_store.yaml
offline_store:
    type: file
    path: data/offline_store
```

### 3. Set Up Data Sources
```python
# example_repo.py
from feast import Entity, Feature, FeatureView, ValueType
from feast.data_source import FileSource
from datetime import timedelta

# Define entity
driver = Entity(
    name="driver_id",
    value_type=ValueType.INT64
)

# Define feature view
driver_stats_fv = FeatureView(
    name="driver_stats",
    entities=["driver_id"],
    ttl=timedelta(days=1),
    features=[
        Feature(name="avg_daily_trips", dtype=ValueType.FLOAT),
    ],
    source=FileSource(
        path="data/driver_stats.parquet"
    )
)
```

## ✅ Verification

### Test Feast Installation
```bash
# Check Feast version
feast --version

# Expected output: 
# Feast SDK version: 0.x.x
```

### Test Basic Functionality
```python
# test_feast.py
from feast import FeatureStore

# Initialize feature store
fs = FeatureStore(repo_path=".")

# Test feature store
print(f"Feature store initialized: {fs.config.project}")

# List feature views
feature_views = fs.list_feature_views()
print(f"Feature views: {[fv.name for fv in feature_views]}")
```

### Test Online Store
```bash
# Start Redis (if using Redis)
docker run -d -p 6379:6379 redis:alpine

# Materialize features
feast materialize-incremental $(date +%Y-%m-%d)

# Test online retrieval
python -c "
from feast import FeatureStore
fs = FeatureStore(repo_path='.')
features = fs.get_online_features(
    features=['driver_stats:avg_daily_trips'],
    entity_rows=[{'driver_id': 1001}]
)
print(features.to_dict())
"
```

## 🐛 Common Installation Issues

### Issue 1: Permission Denied
```bash
# Error: Permission denied when installing
# Solution: Use --user flag
pip install --user feast[all]
```

### Issue 2: Python Version Incompatibility
```bash
# Error: Python version not supported
# Solution: Use Python 3.7+
python --version  # Check version
python3.8 -m pip install feast  # Use specific version
```

### Issue 3: Missing Dependencies
```bash
# Error: Missing required dependencies
# Solution: Install with extras
pip install feast[all]
```

### Issue 4: Online Store Connection Issues
```bash
# Error: Cannot connect to online store
# Solution: Check online store configuration
# For Redis:
redis-cli ping  # Should return PONG

# For PostgreSQL:
psql -h localhost -U feast -d feast
```

### Issue 5: Registry Database Issues
```bash
# Error: Registry database not found
# Solution: Initialize registry
feast apply
```

## 🔍 Troubleshooting Commands

### Check Installation
```bash
# Check Feast installation
pip show feast

# Check installed providers
pip list | grep feast

# Check configuration
feast config
```

### Check Feature Store
```bash
# List feature views
feast feature-views list

# Describe feature view
feast feature-views describe driver_stats

# Check registry
feast registry-dump
```

### Check Online Store
```bash
# Test Redis connection
redis-cli ping

# Test PostgreSQL connection
psql -h localhost -U feast -d feast -c "SELECT 1"
```

## 🚀 Next Steps

After successful installation:

1. **📖 Read**: [Configuration Guide](README_Configuration.md)
2. **🛠️ Setup**: Configure your first feature store
3. **📊 Practice**: Create your first feature view
4. **🔄 Learn**: Materialize and serve features

---

*Feast is now ready for your feature store operations! 🎉*