# DataRobot — Best Practices

## 🎯 Production Best Practices

### 1. **Project Organization**
```
datarobot_projects/
├── projects/
│   ├── customer_churn/
│   ├── fraud_detection/
│   └── demand_forecasting/
├── data/
│   ├── raw/
│   ├── processed/
│   └── predictions/
├── models/
└── deployments/
```

### 2. **Project Naming Convention**
```python
# Consistent project naming
# Format: {domain}_{purpose}_{version}
# Examples:
# - customer_churn_v1
# - fraud_detection_v2
# - demand_forecast_v1

project = dr.Project.create(
    project_name="customer_churn_v1",
    sourcedata="data/customers.csv"
)
```

## 🔐 Security Best Practices

### 1. **API Token Management**
```python
# Secure API token handling
import os
from getpass import getpass
import datarobot as dr

# Get API token securely
api_token = os.getenv("DATAROBOT_API_TOKEN")
if not api_token:
    api_token = getpass("Enter your DataRobot API token: ")

# Initialize client
client = dr.Client(
    token=api_token,
    endpoint="https://app.datarobot.com/api/v2"
)
```

### 2. **Access Control**
```python
# Implement project-level access control
project = dr.Project.get(project_id)

# Set project permissions
project.set_permissions(
    users=["data-scientist@example.com"],
    roles=["project_owner"]
)
```

## 📊 Data Management Best Practices

### 1. **Data Quality Validation**
```python
# Validate data before creating project
import pandas as pd
from great_expectations import dataset

def validate_data(data_path):
    """Validate data quality"""
    df = pd.read_csv(data_path)
    ge_df = dataset.PandasDataset(df)
    
    # Check for missing values
    ge_df.expect_column_values_to_not_be_null("target")
    
    # Validate
    validation_result = ge_df.validate()
    
    if not validation_result.success:
        raise ValueError("Data quality validation failed")
    
    return validation_result

# Use before creating project
validate_data("data/train.csv")
project = dr.Project.create(project_name="my-project", sourcedata="data/train.csv")
```

### 2. **Feature Engineering Best Practices**
```python
# Configure feature engineering
project.set_target(
    target="target_column",
    feature_engineering_options={
        "enable_feature_discovery": True,
        "max_feature_derivation_time": 3600
    }
)
```

## 🔄 CI/CD Integration

### 1. **Automated Model Training**
```yaml
# .github/workflows/datarobot-ci-cd.yml
name: DataRobot CI/CD

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
    
    - name: Install DataRobot
      run: pip install datarobot
    
    - name: Run AutoML
      env:
        DATAROBOT_API_TOKEN: ${{ secrets.DATAROBOT_API_TOKEN }}
      run: |
        python train_model.py
```

## 📈 Monitoring Best Practices

### 1. **Model Performance Monitoring**
```python
# Monitor model performance
def monitor_model_performance(deployment):
    """Monitor deployed model performance"""
    metrics = deployment.get_performance_metrics()
    
    # Check accuracy
    if metrics.get("accuracy", 0) < 0.8:
        send_alert("Model accuracy below threshold")
    
    # Check latency
    if metrics.get("latency", 0) > 100:  # 100ms
        send_alert("Model latency too high")
    
    return metrics
```

### 2. **Data Drift Detection**
```python
# Monitor data drift
deployment = dr.Deployment.get(deployment_id)

# Get monitoring data
monitoring_data = deployment.get_association_ids()

# Check for data drift
if monitoring_data.get("drift_detected", False):
    send_alert("Data drift detected in production")
```

## ⚡ Performance Optimization

### 1. **Model Selection**
```python
# Select best model based on multiple criteria
models = project.get_models()

# Filter by accuracy and latency
best_models = [
    m for m in models
    if m.metrics['AUC'] > 0.8 and m.metrics.get('latency', 0) < 100
]

# Select best model
best_model = max(best_models, key=lambda x: x.metrics['AUC'])
```

### 2. **Batch Prediction Optimization**
```python
# Optimize batch predictions
job = dr.BatchPredictionJob.score(
    deployment_id=deployment.id,
    intake_settings={
        'type': 'local_file',
        'file': 'data/batch.csv'
    },
    output_settings={
        'type': 'local_file',
        'path': 'data/predictions.csv'
    },
    max_explanations=10  # Limit explanations for performance
)
```

## 🔧 Reproducibility Best Practices

### 1. **Version Control**
```bash
# Version control projects and models
git init
git add projects/ models/
git commit -m "Add DataRobot projects v1.0"

# Tag versions
git tag v1.0.0
```

### 2. **Model Versioning**
```python
# Proper model versioning
project = dr.Project.create(
    project_name="model_v1",
    sourcedata="data/train.csv"
)

# Tag model versions
best_model = project.get_models()[0]
best_model.set_tag("version", "1.0.0")
best_model.set_tag("status", "production")
```

## 🧪 Testing Best Practices

### 1. **Model Validation**
```python
# Validate model before deployment
def validate_model(model):
    """Validate model before deployment"""
    # Check accuracy
    if model.metrics['AUC'] < 0.8:
        raise ValueError("Model accuracy too low")
    
    # Check other metrics
    if model.metrics.get('precision', 0) < 0.7:
        raise ValueError("Model precision too low")
    
    return True

# Use before deployment
validate_model(best_model)
deployment = dr.Deployment.create_from_learning_model(model_id=best_model.id)
```

## 📚 Documentation Best Practices

### 1. **Project Documentation**
```python
# Comprehensive project documentation
project = dr.Project.create(
    project_name="customer_churn",
    sourcedata="data/customers.csv",
    description="""
    Customer churn prediction project.
    
    Target: churn (binary classification)
    Features: customer demographics, usage patterns, support interactions
    
    Business goal: Reduce churn by 20%
    """
)
```

## 🚨 Common Pitfalls to Avoid

### 1. **Don't Ignore Model Validation**
```python
# Bad: Deploy without validation
deployment = dr.Deployment.create_from_learning_model(model_id=model.id)

# Good: Validate before deployment
validate_model(model)
if model.metrics['AUC'] > 0.8:
    deployment = dr.Deployment.create_from_learning_model(model_id=model.id)
```

### 2. **Don't Skip Monitoring**
```python
# Bad: No monitoring
deployment = dr.Deployment.create_from_learning_model(model_id=model.id)

# Good: Set up monitoring
deployment = dr.Deployment.create_from_learning_model(model_id=model.id)
setup_monitoring(deployment)
```

### 3. **Don't Hardcode Configuration**
```python
# Bad: Hardcoded API token
client = dr.Client(token="hardcoded-token")

# Good: Use environment variables
client = dr.Client(token=os.getenv("DATAROBOT_API_TOKEN"))
```

---

*Follow these best practices to build robust ML systems with DataRobot! 🎯*