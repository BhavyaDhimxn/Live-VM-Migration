# Airflow — Installation

## 🚀 Installation Methods

Apache Airflow can be installed using multiple methods depending on your environment and needs.

## 📦 Method 1: pip (Recommended)

### Basic Installation
```bash
# Install Airflow
pip install apache-airflow

# Install with specific providers
pip install apache-airflow[amazon]      # AWS providers
pip install apache-airflow[google]     # Google Cloud providers
pip install apache-airflow[azure]      # Azure providers
pip install apache-airflow[kubernetes] # Kubernetes executor
pip install apache-airflow[all]        # All providers
```

### Virtual Environment (Recommended)
```bash
# Create virtual environment
python -m venv airflow-env
source airflow-env/bin/activate  # On Windows: airflow-env\Scripts\activate

# Install Airflow
pip install apache-airflow[all]
```

## 🐳 Method 2: Docker

### Docker Compose (Recommended for Development)
```bash
# Download docker-compose.yaml
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.7.0/docker-compose.yaml'

# Initialize database
docker-compose up airflow-init

# Start Airflow
docker-compose up -d

# Access Airflow UI at http://localhost:8080
# Default credentials: admin/admin
```

### Custom Docker Image
```dockerfile
# Dockerfile
FROM apache/airflow:2.7.0

# Install additional dependencies
USER root
RUN apt-get update && apt-get install -y \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

USER airflow
RUN pip install --no-cache-dir \
    apache-airflow[amazon,google,kubernetes]

# Copy DAGs
COPY dags/ ${AIRFLOW_HOME}/dags/
COPY config/ ${AIRFLOW_HOME}/config/
```

## ☁️ Method 3: Cloud Installation

### AWS (Managed Workflows for Apache Airflow)
```bash
# Install AWS CLI
pip install awscli

# Create MWAA environment via AWS Console or CLI
aws mwaa create-environment \
    --name my-airflow-env \
    --execution-role-arn arn:aws:iam::123456789012:role/mwaa-role \
    --source-bucket-arn arn:aws:s3:::my-airflow-bucket \
    --dag-s3-path dags/
```

### Google Cloud (Cloud Composer)
```bash
# Install gcloud CLI
curl https://sdk.cloud.google.com | bash

# Create Composer environment
gcloud composer environments create my-airflow-env \
    --location us-central1 \
    --python-version 3 \
    --image-version composer-2.0.0-airflow-2.7.0
```

### Azure (Azure Data Factory)
```bash
# Install Azure CLI
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Create Data Factory
az datafactory create \
    --resource-group myResourceGroup \
    --factory-name my-airflow-factory \
    --location eastus
```

## 📋 Prerequisites

### System Requirements
- **Python**: 3.8+ (recommended 3.9+)
- **Memory**: 4GB+ RAM (8GB+ recommended)
- **Storage**: 10GB+ free space
- **Database**: PostgreSQL, MySQL, or SQLite (for development)

### Required Tools
```bash
# Install Python
python --version  # Should be 3.8+

# Install pip
pip --version

# Install Git (for DAG version control)
git --version

# Install Docker (optional, for containerized deployment)
docker --version
```

## ⚙️ Installation Steps

### Step 1: Install Airflow
```bash
# Install Airflow with all providers
pip install apache-airflow[all]

# Verify installation
airflow version
```

### Step 2: Initialize Database
```bash
# Set Airflow home directory
export AIRFLOW_HOME=~/airflow

# Initialize database
airflow db init

# Create admin user
airflow users create \
    --username admin \
    --firstname Admin \
    --lastname User \
    --role Admin \
    --email admin@example.com \
    --password admin
```

### Step 3: Start Airflow
```bash
# Start scheduler (in one terminal)
airflow scheduler

# Start webserver (in another terminal)
airflow webserver --port 8080

# Access UI at http://localhost:8080
```

## 🔧 Post-Installation Setup

### 1. Configure Airflow
```bash
# Create airflow.cfg
airflow config get-value core dags_folder

# Edit configuration
nano ~/airflow/airflow.cfg
```

### 2. Set Up DAGs Directory
```bash
# Create DAGs directory
mkdir -p ~/airflow/dags

# Create sample DAG
cat > ~/airflow/dags/sample_dag.py << 'EOF'
from airflow import DAG
from airflow.operators.bash import BashOperator
from datetime import datetime

dag = DAG(
    'sample_dag',
    start_date=datetime(2023, 1, 1),
    schedule_interval='@daily',
    catchup=False
)

task = BashOperator(
    task_id='print_date',
    bash_command='date',
    dag=dag
)
EOF
```

### 3. Configure Executor
```bash
# For SequentialExecutor (default, single task at a time)
# No additional configuration needed

# For LocalExecutor (multiple tasks)
# Edit airflow.cfg:
# executor = LocalExecutor

# For CeleryExecutor (distributed)
# Install Redis/RabbitMQ and configure in airflow.cfg
```

## ✅ Verification

### Test Airflow Installation
```bash
# Check Airflow version
airflow version

# Expected output: 
# 2.7.0
```

### Test Basic Functionality
```bash
# List DAGs
airflow dags list

# Test DAG
airflow dags test sample_dag 2023-01-01

# Check DAG syntax
python ~/airflow/dags/sample_dag.py
```

### Test Web UI
```bash
# Start webserver
airflow webserver --port 8080

# Access UI at http://localhost:8080
# Login with admin/admin (or your credentials)
```

## 🐛 Common Installation Issues

### Issue 1: Permission Denied
```bash
# Error: Permission denied when installing
# Solution: Use --user flag
pip install --user apache-airflow[all]
```

### Issue 2: Python Version Incompatibility
```bash
# Error: Python version not supported
# Solution: Use Python 3.8+
python --version  # Check version
python3.9 -m pip install apache-airflow  # Use specific version
```

### Issue 3: Database Connection Issues
```bash
# Error: Cannot connect to database
# Solution: Check database configuration
# For PostgreSQL:
psql -h localhost -U airflow -d airflow

# For MySQL:
mysql -h localhost -u airflow -p airflow
```

### Issue 4: Missing Dependencies
```bash
# Error: Missing required dependencies
# Solution: Install with extras
pip install apache-airflow[all]
```

### Issue 5: Port Already in Use
```bash
# Error: Port 8080 already in use
# Solution: Use different port
airflow webserver --port 8081
```

## 🔍 Troubleshooting Commands

### Check Installation
```bash
# Check Airflow installation
pip show apache-airflow

# Check installed providers
airflow providers list

# Check configuration
airflow config list
```

### Check Database
```bash
# Check database connection
airflow db check

# Upgrade database
airflow db upgrade

# Reset database (WARNING: deletes all data)
airflow db reset
```

### Check DAGs
```bash
# List DAGs
airflow dags list

# Show DAG structure
airflow dags show sample_dag

# Test DAG
airflow dags test sample_dag 2023-01-01
```

## 🚀 Next Steps

After successful installation:

1. **📖 Read**: [Configuration Guide](README_Configuration.md)
2. **🛠️ Setup**: Configure your first DAG
3. **📊 Practice**: Create your first workflow
4. **🔄 Learn**: Explore Airflow operators and sensors

---

*Airflow is now ready for your workflow orchestration! 🎉*