# Hopsworks — Best Practices

## 🎯 Production Best Practices

### 1. **Project Organization**
```
hopsworks_project/
├── feature_groups/
│   ├── driver_features.py
│   ├── customer_features.py
│   └── product_features.py
├── models/
│   ├── training_pipeline.py
│   ├── serving.py
│   └── evaluation.py
├── pipelines/
│   ├── data_pipeline.py
│   └── ml_pipeline.py
└── notebooks/
    ├── exploration.ipynb
    └── analysis.ipynb
```

### 2. **Feature Group Naming Convention**
```python
# Consistent feature group naming
# Format: {entity}_{purpose}_{version}
# Examples:
# - driver_stats_v1
# - customer_behavior_v2
# - product_features_v1

fg = fs.create_feature_group(
    name="driver_stats_v1",
    version=1,
    description="Driver statistics features version 1"
)
```

## 🔐 Security Best Practices

### 1. **API Key Management**
```python
# Secure API key handling
import os
from getpass import getpass
import hopsworks

# Get API key securely
api_key = os.getenv("HOPSWORKS_API_KEY")
if not api_key:
    api_key = getpass("Enter your Hopsworks API key: ")

# Initialize connection
project = hopsworks.login(
    api_key_value=api_key,
    project="my_project"
)
```

### 2. **Access Control**
```python
# Implement project-level access control
import hopsworks

project = hopsworks.login(
    api_key_value="your-api-key",
    project="my_project"
)

# Set project permissions
project.set_permissions(
    users=["data-scientist@example.com"],
    role="data_scientist"
)

# Set feature group permissions
fg = fs.get_feature_group("driver_features", version=1)
fg.set_permissions(
    users=["ml-engineer@example.com"],
    role="ml_engineer"
)
```

## 📊 Data Management Best Practices

### 1. **Feature Group Versioning**
```python
# Use semantic versioning for feature groups
fg_v1 = fs.create_feature_group(
    name="driver_features",
    version=1,
    tags={"status": "stable", "version": "1.0.0"}
)

fg_v2 = fs.create_feature_group(
    name="driver_features",
    version=2,
    tags={"status": "beta", "version": "2.0.0"}
)
```

### 2. **Data Quality Validation**
```python
# Implement data quality checks
from great_expectations import dataset

def validate_feature_group(fg, data):
    """Validate feature group data quality"""
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
        raise ValueError("Feature group data quality validation failed")
    
    return validation_result

# Use in feature group creation
fg = fs.create_feature_group("driver_features", version=1)
validate_feature_group(fg, data)
fg.insert(data)
```

### 3. **Feature Documentation**
```python
# Comprehensive feature documentation
fg = fs.create_feature_group(
    name="driver_features",
    version=1,
    description="""
    Driver statistics features for ML models.
    
    Features:
    - avg_daily_trips: Average number of trips per day
    - total_trips: Total number of trips completed
    
    Update frequency: Daily
    Data source: Trip database
    """,
    tags={
        "team": "ml-team",
        "owner": "data-scientist@example.com",
        "documentation": "https://docs.example.com/features/driver"
    }
)
```

## 🔄 CI/CD Integration

### 1. **Automated Model Training**
```yaml
# .github/workflows/hopsworks-ci-cd.yml
name: Hopsworks CI/CD

on:
  push:
    branches: [main]

jobs:
  train:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    
    - name: Setup Python
      uses: actions/setup-python@v2
      with:
        python-version: 3.9
    
    - name: Install dependencies
      run: |
        pip install hopsworks[all]
    
    - name: Run training pipeline
      env:
        HOPSWORKS_API_KEY: ${{ secrets.HOPSWORKS_API_KEY }}
      run: |
        python training_pipeline.py
```

### 2. **Automated Model Deployment**
```python
# automated_deployment.py
import hsml
import os

def deploy_model_automated(model_name, version):
    """Automated model deployment"""
    mr = hsml.connection().get_model_registry()
    model = mr.get_model(model_name, version=version)
    
    # Check if model meets deployment criteria
    metrics = model.get_metrics()
    if metrics["accuracy"] < 0.8:
        raise ValueError("Model accuracy below threshold")
    
    # Deploy model
    deployment = model.deploy(
        name=f"{model_name}_production",
        script_file="serving.py"
    )
    
    return deployment
```

## 📈 Monitoring Best Practices

### 1. **Model Performance Monitoring**
```python
# Monitor model performance
def monitor_model_performance(deployment):
    """Monitor deployed model performance"""
    metrics = deployment.get_metrics()
    
    # Check accuracy
    if metrics["accuracy"] < 0.8:
        send_alert("Model accuracy below threshold")
    
    # Check latency
    if metrics["latency"] > 100:  # 100ms
        send_alert("Model latency too high")
    
    return metrics
```

### 2. **Feature Monitoring**
```python
# Monitor feature quality
def monitor_feature_quality(fg):
    """Monitor feature group quality"""
    features = fg.read()
    
    # Check for missing values
    missing_pct = features.isnull().sum() / len(features)
    if missing_pct.any() > 0.1:  # 10% threshold
        send_alert("High missing value percentage in features")
    
    # Check for outliers
    for col in features.columns:
        if features[col].dtype in ['float64', 'int64']:
            mean = features[col].mean()
            std = features[col].std()
            outliers = features[abs(features[col] - mean) > 3 * std]
            if len(outliers) > 0:
                send_alert(f"Outliers detected in {col}")
```

## ⚡ Performance Optimization

### 1. **Feature Group Optimization**
```python
# Optimize feature group reads
fg = fs.get_feature_group("driver_features", version=1)

# Use filters for efficient reads
features = fg.read(
    filter=fg.avg_daily_trips > 10,
    limit=1000  # Limit results
)

# Use specific columns
features = fg.read(
    columns=["driver_id", "avg_daily_trips"]
)
```

### 2. **Pipeline Optimization**
```python
# Optimize pipeline execution
@pipeline(name="optimized_pipeline")
def optimized_pipeline():
    """Optimized ML pipeline"""
    # Use caching for expensive operations
    @cache
    def expensive_operation():
        # Expensive computation
        pass
    
    # Parallel processing
    from multiprocessing import Pool
    with Pool(processes=4) as pool:
        results = pool.map(process_data, data_chunks)
    
    return results
```

## 🔧 Reproducibility Best Practices

### 1. **Version Control**
```bash
# Version control feature groups and models
git init
git add feature_groups/ models/
git commit -m "Add driver features and model v1.0"

# Tag versions
git tag v1.0.0
```

### 2. **Environment Management**
```python
# Use consistent environments
# requirements.txt
hopsworks==3.0.0
pandas==1.5.3
scikit-learn==1.2.0

# Use virtual environments
python -m venv hopsworks-env
source hopsworks-env/bin/activate
pip install -r requirements.txt
```

### 3. **Model Versioning**
```python
# Proper model versioning
mr = hsml.connection().get_model_registry()

# Create versioned model
model = mr.python.create_model(
    name="driver_prediction_model",
    version=1,
    description="Driver prediction model v1.0",
    tags={"version": "1.0.0", "status": "production"}
)
```

## 🧪 Testing Best Practices

### 1. **Unit Tests**
```python
# test_feature_groups.py
import pytest
import hsfs

def test_feature_group_creation():
    """Test feature group creation"""
    connection = hsfs.connection()
    fs = connection.get_feature_store("my_feature_store")
    
    fg = fs.create_feature_group(
        name="test_features",
        version=1
    )
    
    assert fg.name == "test_features"
    assert fg.version == 1

def test_feature_group_read():
    """Test feature group read"""
    fg = fs.get_feature_group("test_features", version=1)
    features = fg.read()
    
    assert features is not None
    assert len(features) > 0
```

### 2. **Integration Tests**
```python
# test_model_lifecycle.py
import pytest
import hsml

def test_model_lifecycle():
    """Test complete model lifecycle"""
    mr = hsml.connection().get_model_registry()
    
    # Create model
    model = mr.python.create_model(
        name="test_model",
        version=1
    )
    
    # Save model
    model.save("test_model.pkl")
    
    # Deploy model
    deployment = model.deploy(
        name="test_deployment",
        script_file="serving.py"
    )
    
    assert deployment.status == "running"
```

## 📚 Documentation Best Practices

### 1. **Feature Group Documentation**
```python
# Comprehensive feature group documentation
fg = fs.create_feature_group(
    name="driver_features",
    version=1,
    description="""
    # Driver Features
    
    ## Overview
    This feature group contains driver statistics features.
    
    ## Features
    - avg_daily_trips: Average trips per day
    - total_trips: Total trips completed
    
    ## Usage
    ```python
    fg = fs.get_feature_group("driver_features", version=1)
    features = fg.read()
    ```
    """,
    tags={
        "documentation": "https://docs.example.com/features/driver",
        "owner": "ml-team@example.com"
    }
)
```

## 🚨 Common Pitfalls to Avoid

### 1. **Don't Ignore Feature Versioning**
```python
# Bad: No versioning
fg = fs.create_feature_group("driver_features")

# Good: Versioned feature groups
fg = fs.create_feature_group("driver_features", version=1)
```

### 2. **Don't Skip Data Validation**
```python
# Bad: No validation
fg.insert(data)

# Good: Validate before insert
validate_data(data)
fg.insert(data)
```

### 3. **Don't Hardcode Configuration**
```python
# Bad: Hardcoded API key
project = hopsworks.login(api_key_value="hardcoded-key")

# Good: Use environment variables
project = hopsworks.login(
    api_key_value=os.getenv("HOPSWORKS_API_KEY")
)
```

---

*Follow these best practices to build robust, scalable ML systems with Hopsworks! 🎯*