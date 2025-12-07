# Feast — Configuration

## ⚙️ Configuration Overview

Feast configuration is managed through the `feature_store.yaml` file, which defines the feature store project, registry, providers, and storage backends.

## 📁 Configuration Files

### 1. **Feature Store Configuration** (`feature_store.yaml`)
```yaml
# Basic configuration
project: my_feature_store
registry: data/registry.db
provider: local

# Online store configuration
online_store:
    type: redis
    connection_string: "localhost:6379"

# Offline store configuration
offline_store:
    type: file
    path: data/offline_store
```

### 2. **Environment Variables**
```bash
# Set feature store path
export FEAST_REPO_PATH="/path/to/feast_repo"

# Set registry path
export FEAST_REGISTRY_PATH="/path/to/registry.db"
```

## 🔧 Basic Configuration

### 1. **Project Configuration**
```yaml
# feature_store.yaml
project: my_feature_store
registry: data/registry.db
provider: local
```

### 2. **Registry Configuration**
```yaml
# Local registry (SQLite)
registry: data/registry.db

# PostgreSQL registry
registry:
    path: postgresql://user:password@localhost:5432/feast
    cache_ttl_seconds: 60

# S3 registry
registry:
    path: s3://my-bucket/feast/registry.db
    cache_ttl_seconds: 60
```

### 3. **Provider Configuration**
```yaml
# Local provider
provider: local

# AWS provider
provider: aws
online_store:
    type: dynamodb
    region: us-west-2

# GCP provider
provider: gcp
online_store:
    type: bigtable
    project_id: my-project
    instance_id: my-instance

# Azure provider
provider: azure
online_store:
    type: cosmosdb
    account_key: your-account-key
```

## ☁️ Online Store Configuration

### 1. **Redis Configuration**
```yaml
online_store:
    type: redis
    connection_string: "localhost:6379"
    # Or with password
    connection_string: "redis://:password@localhost:6379"
    # Or with SSL
    connection_string: "rediss://localhost:6380"
```

### 2. **PostgreSQL Configuration**
```yaml
online_store:
    type: postgres
    host: localhost
    port: 5432
    database: feast
    user: feast
    password: feast
    # Or connection string
    connection_string: "postgresql://user:password@localhost:5432/feast"
```

### 3. **DynamoDB Configuration (AWS)**
```yaml
online_store:
    type: dynamodb
    region: us-west-2
    table_name: feast-online-store
    # Optional: use existing AWS credentials
    # aws_access_key_id: your-access-key
    # aws_secret_access_key: your-secret-key
```

### 4. **Bigtable Configuration (GCP)**
```yaml
online_store:
    type: bigtable
    project_id: my-project
    instance_id: my-instance
    # Optional: credentials path
    # credentials_path: /path/to/credentials.json
```

## 📊 Offline Store Configuration

### 1. **File-based Offline Store**
```yaml
offline_store:
    type: file
    path: data/offline_store
```

### 2. **BigQuery Offline Store (GCP)**
```yaml
offline_store:
    type: bigquery
    project: my-project
    dataset: feast_dataset
    # Optional: credentials path
    # credentials_path: /path/to/credentials.json
```

### 3. **Redshift Offline Store (AWS)**
```yaml
offline_store:
    type: redshift
    region: us-west-2
    cluster_id: my-cluster
    database: feast
    user: feast
    password: feast
    s3_staging_location: s3://my-bucket/staging
```

### 4. **Snowflake Offline Store**
```yaml
offline_store:
    type: snowflake
    account: my-account
    user: feast
    password: feast
    role: feast_role
    warehouse: feast_warehouse
    database: feast_db
    schema: feast_schema
```

## 🔐 Security Configuration

### 1. **Authentication Configuration**
```yaml
# AWS authentication
online_store:
    type: dynamodb
    region: us-west-2
    # Use IAM roles or credentials
    aws_access_key_id: ${AWS_ACCESS_KEY_ID}
    aws_secret_access_key: ${AWS_SECRET_ACCESS_KEY}

# GCP authentication
online_store:
    type: bigtable
    project_id: my-project
    credentials_path: ${GOOGLE_APPLICATION_CREDENTIALS}
```

### 2. **Connection Security**
```yaml
# Secure Redis connection
online_store:
    type: redis
    connection_string: "rediss://:password@localhost:6380"
    ssl_ca_certs: /path/to/ca.crt

# Secure PostgreSQL connection
online_store:
    type: postgres
    connection_string: "postgresql://user:password@localhost:5432/feast?sslmode=require"
```

## 🔄 Advanced Configuration

### 1. **Feature View Configuration**
```python
# example_repo.py
from feast import Entity, Feature, FeatureView, ValueType
from feast.data_source import FileSource
from datetime import timedelta

# Entity definition
driver = Entity(
    name="driver_id",
    value_type=ValueType.INT64,
    description="Driver identifier"
)

# Feature view with TTL
driver_stats_fv = FeatureView(
    name="driver_stats",
    entities=["driver_id"],
    ttl=timedelta(days=1),  # Features expire after 1 day
    features=[
        Feature(name="avg_daily_trips", dtype=ValueType.FLOAT),
        Feature(name="total_trips", dtype=ValueType.INT64),
    ],
    source=FileSource(
        path="data/driver_stats.parquet",
        timestamp_field="event_timestamp"
    ),
    online=True,  # Enable online serving
    tags={"team": "ml-team", "version": "v1"}
)
```

### 2. **Data Source Configuration**
```python
# File source
file_source = FileSource(
    path="data/driver_stats.parquet",
    timestamp_field="event_timestamp",
    created_timestamp_column="created_at"
)

# BigQuery source
from feast.data_source import BigQuerySource

bq_source = BigQuerySource(
    table="my-project.dataset.driver_stats",
    timestamp_field="event_timestamp"
)

# Kafka source (streaming)
from feast.data_source import KafkaSource

kafka_source = KafkaSource(
    kafka_bootstrap_servers="localhost:9092",
    topic="driver_stats",
    message_format="avro",
    timestamp_field="event_timestamp"
)
```

### 3. **Materialization Configuration**
```yaml
# Materialization settings
materialization:
    enabled: true
    schedule: "0 * * * *"  # Hourly
    batch_size: 10000
    max_workers: 4
```

## 🐛 Common Configuration Issues

### Issue 1: Registry Connection Failed
```bash
# Error: Cannot connect to registry
# Solution: Check registry configuration
# For PostgreSQL:
psql -h localhost -U feast -d feast

# For S3:
aws s3 ls s3://my-bucket/feast/registry.db
```

### Issue 2: Online Store Connection Issues
```bash
# Error: Cannot connect to online store
# Solution: Check online store configuration
# For Redis:
redis-cli ping

# For PostgreSQL:
psql -h localhost -U feast -d feast -c "SELECT 1"
```

### Issue 3: Offline Store Access Denied
```bash
# Error: Access denied to offline store
# Solution: Check credentials and permissions
# For BigQuery:
gcloud auth application-default login

# For S3:
aws s3 ls s3://my-bucket/
```

### Issue 4: Feature View Not Found
```bash
# Error: Feature view not found
# Solution: Apply feature definitions
feast apply

# Verify feature views
feast feature-views list
```

## ✅ Configuration Validation

### Test Configuration
```bash
# Validate feature store configuration
feast config

# Test registry connection
feast registry-dump

# Test online store connection
python -c "
from feast import FeatureStore
fs = FeatureStore(repo_path='.')
print('Feature store configured successfully')
"
```

### Configuration Checklist
- [ ] Feature store project configured
- [ ] Registry configured and accessible
- [ ] Online store configured and accessible
- [ ] Offline store configured and accessible
- [ ] Data sources configured
- [ ] Feature views defined
- [ ] Authentication configured (if needed)
- [ ] Materialization configured (if needed)

## 🔧 Configuration Best Practices

1. **🔐 Security**: Use environment variables for credentials
2. **💾 Registry**: Use PostgreSQL for production registry
3. **🚀 Online Store**: Use Redis for low-latency serving
4. **📊 Offline Store**: Use cloud data warehouses for scale
5. **📝 Documentation**: Document all feature definitions
6. **🧪 Testing**: Test configuration in development first

---

*Your Feast configuration is now optimized for your feature store! 🎯*