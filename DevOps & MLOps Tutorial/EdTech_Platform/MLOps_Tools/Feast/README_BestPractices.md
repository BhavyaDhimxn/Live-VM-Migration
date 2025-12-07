# Feast — Best Practices

## 🎯 Production Best Practices

### 1. **Feature Store Organization**
```
feast_repo/
├── feature_store.yaml
├── driver_features.py
├── customer_features.py
├── product_features.py
├── data/
│   ├── raw/
│   ├── processed/
│   └── registry.db
└── tests/
    ├── test_driver_features.py
    └── test_customer_features.py
```

### 2. **Feature Naming Convention**
```python
# Consistent feature naming
# Format: {entity}_{metric}_{aggregation}
# Examples:
# - driver_avg_daily_trips
# - customer_total_orders
# - product_avg_rating

driver_avg_daily_trips = Feature(
    name="avg_daily_trips",
    dtype=ValueType.FLOAT
)
```

## 🔐 Security Best Practices

### 1. **Credential Management**
```python
# Use environment variables for credentials
import os

# feature_store.yaml
online_store:
    type: dynamodb
    region: us-west-2
    aws_access_key_id: ${AWS_ACCESS_KEY_ID}
    aws_secret_access_key: ${AWS_SECRET_ACCESS_KEY}
```

### 2. **Access Control**
```python
# Implement feature-level access control
def get_features_with_access_control(user_role, features):
    """Get features based on user role"""
    allowed_features = {
        "admin": ["*"],  # All features
        "scientist": ["driver_stats:*", "customer_stats:*"],
        "viewer": ["driver_stats:avg_daily_trips"]  # Limited features
    }
    
    if user_role not in allowed_features:
        raise ValueError(f"Invalid user role: {user_role}")
    
    # Filter features based on role
    user_features = allowed_features[user_role]
    
    return user_features
```

### 3. **Data Privacy**
```python
# Implement PII masking
def mask_pii_features(features):
    """Mask personally identifiable information"""
    pii_fields = ['driver_ssn', 'driver_email', 'driver_phone']
    
    masked_features = features.copy()
    for field in pii_fields:
        if field in masked_features:
            masked_features[field] = "[REDACTED]"
    
    return masked_features
```

## 📊 Data Management Best Practices

### 1. **Feature Versioning Strategy**
```python
# Use semantic versioning for features
driver_stats_v1 = FeatureView(
    name="driver_stats_v1",
    tags={"version": "1.0.0", "status": "stable"}
)

driver_stats_v2 = FeatureView(
    name="driver_stats_v2",
    tags={"version": "2.0.0", "status": "beta"}
)

# Deprecate old versions
driver_stats_v1_deprecated = FeatureView(
    name="driver_stats_v1",
    tags={"version": "1.0.0", "status": "deprecated"}
)
```

### 2. **Data Quality Validation**
```python
# Implement data quality checks
from great_expectations import dataset

def validate_feature_quality(data):
    """Validate feature data quality"""
    ge_df = dataset.PandasDataset(data)
    
    # Check for missing values
    ge_df.expect_column_values_to_not_be_null("avg_daily_trips")
    
    # Check value ranges
    ge_df.expect_column_values_to_be_between(
        "avg_daily_trips", 
        min_value=0, 
        max_value=100
    )
    
    # Validate
    validation_result = ge_df.validate()
    
    if not validation_result.success:
        raise ValueError("Feature data quality validation failed")
    
    return validation_result
```

### 3. **Feature Documentation**
```python
# Comprehensive feature documentation
driver_stats_fv = FeatureView(
    name="driver_stats",
    entities=["driver_id"],
    ttl=timedelta(days=1),
    features=[
        Feature(
            name="avg_daily_trips",
            dtype=ValueType.FLOAT,
            description="Average number of trips per day for the driver"
        ),
        Feature(
            name="total_trips",
            dtype=ValueType.INT64,
            description="Total number of trips completed by the driver"
        ),
    ],
    source=FileSource(
        path="data/driver_stats.parquet",
        timestamp_field="event_timestamp"
    ),
    tags={
        "team": "ml-team",
        "owner": "data-scientist@example.com",
        "documentation": "https://docs.example.com/features/driver_stats"
    }
)
```

## 🔄 CI/CD Integration

### 1. **Automated Feature Testing**
```python
# test_features.py
import pytest
from feast import FeatureStore

def test_feature_definitions():
    """Test feature definitions"""
    fs = FeatureStore(repo_path=".")
    
    # Test feature views exist
    feature_views = fs.list_feature_views()
    assert len(feature_views) > 0
    
    # Test feature retrieval
    features = fs.get_online_features(
        features=["driver_stats:avg_daily_trips"],
        entity_rows=[{"driver_id": 1001}]
    )
    
    assert features is not None

def test_feature_quality():
    """Test feature data quality"""
    fs = FeatureStore(repo_path=".")
    
    # Get historical features
    entity_df = pd.DataFrame({
        "driver_id": [1001],
        "event_timestamp": [datetime.now()]
    })
    
    training_df = fs.get_historical_features(
        entity_df=entity_df,
        features=["driver_stats:avg_daily_trips"]
    )
    
    df = training_df.to_df()
    
    # Validate data quality
    assert not df["avg_daily_trips"].isnull().any()
    assert (df["avg_daily_trips"] >= 0).all()
```

### 2. **Automated Materialization**
```yaml
# .github/workflows/feast-materialize.yml
name: Materialize Features

on:
  schedule:
    - cron: '0 * * * *'  # Hourly
  workflow_dispatch:

jobs:
  materialize:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    
    - name: Setup Python
      uses: actions/setup-python@v2
      with:
        python-version: 3.9
    
    - name: Install Feast
      run: pip install feast[all]
    
    - name: Materialize Features
      run: |
        feast materialize-incremental $(date +%Y-%m-%d)
```

## 📈 Monitoring Best Practices

### 1. **Feature Serving Monitoring**
```python
# Monitor feature serving performance
import time
import logging
from feast import FeatureStore

logger = logging.getLogger(__name__)

def monitor_feature_serving(fs, features, entity_rows):
    """Monitor feature serving with metrics"""
    start_time = time.time()
    
    try:
        # Get features
        result = fs.get_online_features(
            features=features,
            entity_rows=entity_rows
        )
        
        latency = time.time() - start_time
        
        # Log metrics
        logger.info(f"Feature serving latency: {latency*1000:.2f}ms")
        logger.info(f"Features retrieved: {len(features)}")
        
        # Alert if slow
        if latency > 0.1:  # 100ms threshold
            send_alert(f"Slow feature serving: {latency*1000:.2f}ms")
        
        return result
    
    except Exception as e:
        logger.error(f"Feature serving error: {e}")
        raise
```

### 2. **Feature Data Monitoring**
```python
# Monitor feature data quality
def monitor_feature_data_quality(fs):
    """Monitor feature data quality"""
    # Get sample features
    features = fs.get_online_features(
        features=["driver_stats:avg_daily_trips"],
        entity_rows=[{"driver_id": 1001}]
    )
    
    df = features.to_df()
    
    # Check for anomalies
    if df["avg_daily_trips"].isnull().any():
        send_alert("Null values detected in avg_daily_trips")
    
    if (df["avg_daily_trips"] < 0).any():
        send_alert("Negative values detected in avg_daily_trips")
    
    # Check for outliers
    mean = df["avg_daily_trips"].mean()
    std = df["avg_daily_trips"].std()
    
    outliers = df[abs(df["avg_daily_trips"] - mean) > 3 * std]
    if len(outliers) > 0:
        send_alert(f"Outliers detected in avg_daily_trips: {len(outliers)}")
```

### 3. **Materialization Monitoring**
```python
# Monitor materialization jobs
def monitor_materialization(fs):
    """Monitor feature materialization"""
    # Check materialization status
    # (This would require custom implementation or Feast API)
    
    # Monitor materialization latency
    start_time = time.time()
    
    # Run materialization
    # feast materialize-incremental $(date +%Y-%m-%d)
    
    latency = time.time() - start_time
    
    # Log metrics
    logger.info(f"Materialization latency: {latency:.2f}s")
    
    # Alert if slow
    if latency > 3600:  # 1 hour threshold
        send_alert(f"Slow materialization: {latency:.2f}s")
```

## ⚡ Performance Optimization

### 1. **Feature Caching**
```python
# Implement feature caching
from functools import lru_cache
from feast import FeatureStore

@lru_cache(maxsize=1000)
def get_cached_features(driver_id):
    """Cache frequently accessed features"""
    fs = FeatureStore(repo_path=".")
    
    features = fs.get_online_features(
        features=["driver_stats:avg_daily_trips"],
        entity_rows=[{"driver_id": driver_id}]
    )
    
    return features.to_dict()
```

### 2. **Batch Feature Retrieval**
```python
# Batch feature retrieval for efficiency
def get_features_batch(fs, driver_ids, features):
    """Retrieve features in batch"""
    entity_rows = [{"driver_id": driver_id} for driver_id in driver_ids]
    
    # Get all features at once
    result = fs.get_online_features(
        features=features,
        entity_rows=entity_rows
    )
    
    return result.to_df()
```

### 3. **Optimize TTL Settings**
```python
# Set appropriate TTL for features
# Short TTL for frequently changing features
real_time_features = FeatureView(
    name="real_time_stats",
    ttl=timedelta(hours=1),  # 1 hour TTL
    features=[...]
)

# Long TTL for stable features
stable_features = FeatureView(
    name="stable_stats",
    ttl=timedelta(days=30),  # 30 days TTL
    features=[...]
)
```

## 🔧 Reproducibility Best Practices

### 1. **Feature Store Versioning**
```bash
# Version control feature store
git init
git add feature_store.yaml driver_features.py
git commit -m "Add driver features v1.0"

# Tag versions
git tag v1.0.0
```

### 2. **Feature Definition Management**
```python
# Use consistent feature definitions
# feature_definitions.py
from feast import Entity, Feature, FeatureView, ValueType

# Centralized entity definitions
DRIVER_ENTITY = Entity(
    name="driver_id",
    value_type=ValueType.INT64
)

# Reusable feature definitions
DRIVER_FEATURES = [
    Feature(name="avg_daily_trips", dtype=ValueType.FLOAT),
    Feature(name="total_trips", dtype=ValueType.INT64),
]

# Use in feature views
driver_stats_fv = FeatureView(
    name="driver_stats",
    entities=[DRIVER_ENTITY],
    features=DRIVER_FEATURES,
    ...
)
```

### 3. **Environment Management**
```yaml
# Development
# feature_store.dev.yaml
project: my_feature_store_dev
online_store:
    type: redis
    connection_string: "localhost:6379"

# Production
# feature_store.prod.yaml
project: my_feature_store_prod
online_store:
    type: dynamodb
    region: us-west-2
```

## 🧪 Testing Best Practices

### 1. **Unit Tests**
```python
# test_feature_definitions.py
import pytest
from feast import FeatureStore

def test_feature_view_definition():
    """Test feature view definition"""
    from driver_features import driver_stats_fv
    
    assert driver_stats_fv.name == "driver_stats"
    assert len(driver_stats_fv.features) > 0
    assert "driver_id" in driver_stats_fv.entities

def test_feature_retrieval():
    """Test feature retrieval"""
    fs = FeatureStore(repo_path=".")
    
    features = fs.get_online_features(
        features=["driver_stats:avg_daily_trips"],
        entity_rows=[{"driver_id": 1001}]
    )
    
    assert features is not None
    assert "avg_daily_trips" in features.to_dict()
```

### 2. **Integration Tests**
```python
# test_feature_integration.py
import pytest
from feast import FeatureStore

def test_training_data_preparation():
    """Test training data preparation"""
    fs = FeatureStore(repo_path=".")
    
    entity_df = pd.DataFrame({
        "driver_id": [1001, 1002],
        "event_timestamp": [datetime.now()] * 2
    })
    
    training_df = fs.get_historical_features(
        entity_df=entity_df,
        features=["driver_stats:avg_daily_trips"]
    )
    
    df = training_df.to_df()
    assert len(df) == 2
    assert "avg_daily_trips" in df.columns
```

## 📚 Documentation Best Practices

### 1. **Feature Documentation**
```python
# Comprehensive feature documentation
driver_stats_fv = FeatureView(
    name="driver_stats",
    entities=["driver_id"],
    ttl=timedelta(days=1),
    features=[
        Feature(
            name="avg_daily_trips",
            dtype=ValueType.FLOAT,
            description="""
            Average number of trips per day for the driver.
            
            Calculation: Total trips in last 30 days / 30
            Update frequency: Daily
            Data source: Trip database
            """
        ),
    ],
    source=FileSource(...),
    tags={
        "documentation": "https://docs.example.com/features/driver_stats",
        "owner": "ml-team@example.com",
        "version": "1.0.0"
    }
)
```

### 2. **Feature Store Documentation**
```markdown
# Feature Store Documentation

## Overview
This feature store contains features for driver analytics.

## Features

### driver_stats
- **Entity**: driver_id
- **Features**:
  - avg_daily_trips: Average trips per day
  - total_trips: Total trips completed
- **Update Frequency**: Daily
- **TTL**: 1 day

## Usage Examples
[Include code examples]
```

## 🚨 Common Pitfalls to Avoid

### 1. **Don't Use Short TTL for Stable Features**
```python
# Bad: Short TTL for stable features
stable_features = FeatureView(
    name="stable_stats",
    ttl=timedelta(hours=1)  # Too short
)

# Good: Appropriate TTL
stable_features = FeatureView(
    name="stable_stats",
    ttl=timedelta(days=30)  # Appropriate for stable features
)
```

### 2. **Don't Ignore Feature Versioning**
```python
# Bad: No versioning
driver_stats = FeatureView(name="driver_stats", ...)

# Good: Versioned features
driver_stats_v1 = FeatureView(
    name="driver_stats_v1",
    tags={"version": "1.0.0"}
)
```

### 3. **Don't Hardcode Feature Names**
```python
# Bad: Hardcoded feature names
features = fs.get_online_features(
    features=["driver_stats:avg_daily_trips"],
    ...
)

# Good: Use constants
DRIVER_FEATURES = ["driver_stats:avg_daily_trips"]
features = fs.get_online_features(
    features=DRIVER_FEATURES,
    ...
)
```

---

*Follow these best practices to build robust, scalable feature stores with Feast! 🎯*