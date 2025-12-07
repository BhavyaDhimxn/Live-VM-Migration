# CometML — Best Practices

## 🎯 Production Best Practices

### 1. **Project Organization**
```
ml-project/
├── experiments/
│   ├── data_preparation/
│   ├── model_training/
│   ├── hyperparameter_tuning/
│   └── model_evaluation/
├── models/
│   ├── production/
│   ├── staging/
│   └── development/
├── data/
│   ├── raw/
│   ├── processed/
│   └── external/
├── notebooks/
├── src/
│   ├── data_preparation.py
│   ├── model_training.py
│   └── model_evaluation.py
└── configs/
    ├── development.yaml
    ├── staging.yaml
    └── production.yaml
```

### 2. **Experiment Naming Convention**
```python
# Consistent experiment naming
import comet_ml
from datetime import datetime

def create_experiment(experiment_type: str, model_name: str, version: str):
    """Create experiment with consistent naming"""
    
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    experiment_name = f"{experiment_type}_{model_name}_v{version}_{timestamp}"
    
    experiment = comet_ml.Experiment(
        project_name="ml-project",
        experiment_name=experiment_name,
        tags=[experiment_type, model_name, version]
    )
    
    return experiment

# Usage
experiment = create_experiment("training", "random-forest", "1.0")
```

## 🔐 Security Best Practices

### 1. **API Key Management**
```python
# Secure API key handling
import os
from getpass import getpass
import comet_ml

def get_api_key():
    """Get API key securely"""
    
    # Try environment variable first
    api_key = os.getenv("COMET_API_KEY")
    
    if not api_key:
        # Prompt user for API key
        api_key = getpass("Enter your CometML API key: ")
    
    if not api_key:
        raise ValueError("API key is required")
    
    return api_key

# Initialize experiment with secure API key
experiment = comet_ml.Experiment(api_key=get_api_key())
```

### 2. **Data Privacy Controls**
```python
# Data privacy and sanitization
import comet_ml
import pandas as pd

class PrivacyAwareCometML:
    def __init__(self, api_key: str):
        self.experiment = comet_ml.Experiment(api_key=api_key)
        self.sensitive_fields = ['ssn', 'credit_card', 'email', 'phone']
    
    def log_data_safely(self, data: pd.DataFrame, data_name: str):
        """Log data with privacy controls"""
        
        # Sanitize sensitive data
        sanitized_data = self._sanitize_data(data)
        
        # Log data statistics instead of raw data
        self.experiment.log_other(f"{data_name}_shape", data.shape)
        self.experiment.log_other(f"{data_name}_columns", list(data.columns))
        self.experiment.log_other(f"{data_name}_dtypes", data.dtypes.to_dict())
        
        # Log summary statistics
        summary_stats = data.describe().to_dict()
        self.experiment.log_other(f"{data_name}_summary", summary_stats)
        
        # Log sample data (first 100 rows)
        sample_data = sanitized_data.head(100)
        self.experiment.log_table(sample_data, f"{data_name}_sample")
    
    def _sanitize_data(self, data: pd.DataFrame) -> pd.DataFrame:
        """Remove sensitive information from data"""
        
        sanitized = data.copy()
        
        for field in self.sensitive_fields:
            if field in sanitized.columns:
                sanitized[field] = "[REDACTED]"
        
        return sanitized
```

### 3. **Access Control**
```python
# Implement access control for experiments
import comet_ml

def create_restricted_experiment(project_name: str, user_role: str):
    """Create experiment with role-based access"""
    
    # Define access levels
    access_levels = {
        "admin": ["read", "write", "delete"],
        "scientist": ["read", "write"],
        "viewer": ["read"]
    }
    
    if user_role not in access_levels:
        raise ValueError(f"Invalid user role: {user_role}")
    
    experiment = comet_ml.Experiment(
        project_name=project_name,
        tags=[f"role:{user_role}", "restricted"]
    )
    
    # Log access information
    experiment.log_other("user_role", user_role)
    experiment.log_other("access_levels", access_levels[user_role])
    
    return experiment
```

## 📊 Data Management Best Practices

### 1. **Data Versioning Strategy**
```python
# Implement data versioning with CometML
import comet_ml
import hashlib
import pandas as pd
from datetime import datetime

def log_data_version(data: pd.DataFrame, data_name: str, data_source: str):
    """Log data version and metadata"""
    
    experiment = comet_ml.Experiment()
    
    # Calculate data hash
    data_hash = hashlib.md5(data.to_string().encode()).hexdigest()
    
    # Log data metadata
    experiment.log_other(f"{data_name}_hash", data_hash)
    experiment.log_other(f"{data_name}_shape", data.shape)
    experiment.log_other(f"{data_name}_columns", list(data.columns))
    experiment.log_other(f"{data_name}_source", data_source)
    experiment.log_other(f"{data_name}_timestamp", datetime.now().isoformat())
    
    # Log data quality metrics
    experiment.log_other(f"{data_name}_null_count", data.isnull().sum().to_dict())
    experiment.log_other(f"{data_name}_duplicate_count", data.duplicated().sum())
    
    return data_hash
```

### 2. **Data Quality Validation**
```python
# Implement data quality validation
import comet_ml
import pandas as pd
from great_expectations import dataset

def validate_data_quality(data: pd.DataFrame, expectations_suite: str):
    """Validate data quality and log results"""
    
    experiment = comet_ml.Experiment()
    
    # Load expectations
    ge_df = dataset.PandasDataset(data)
    
    # Run validations
    validation_result = ge_df.validate(expectation_suite=expectations_suite)
    
    # Log validation results
    experiment.log_other("data_quality_score", validation_result.success_rate)
    experiment.log_other("data_validation_result", validation_result.to_json_dict())
    
    if not validation_result.success:
        experiment.log_other("data_quality_issues", "true")
        experiment.log_other("failed_expectations", validation_result.failed_expectations)
        raise ValueError("Data quality validation failed")
    
    return validation_result
```

### 3. **Data Lineage Tracking**
```python
# Track data lineage through CometML
import comet_ml
from typing import Dict, List

def track_data_lineage(
    input_data: Dict[str, str],
    transformations: List[str],
    output_data: Dict[str, str]
):
    """Track data lineage through CometML"""
    
    experiment = comet_ml.Experiment()
    
    # Log input data sources
    for name, path in input_data.items():
        experiment.log_other(f"input_data_{name}", path)
    
    # Log transformations
    experiment.log_other("transformations", transformations)
    
    # Log output data
    for name, path in output_data.items():
        experiment.log_other(f"output_data_{name}", path)
    
    # Log lineage graph
    lineage = {
        "inputs": input_data,
        "transformations": transformations,
        "outputs": output_data,
        "timestamp": datetime.now().isoformat()
    }
    experiment.log_other("data_lineage", lineage)
```

## 🔄 CI/CD Integration

### 1. **Automated Experiment Tracking**
```yaml
# .github/workflows/cometml-ci-cd.yml
name: CometML CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    
    - name: Setup Python
      uses: actions/setup-python@v2
      with:
        python-version: 3.9
    
    - name: Install dependencies
      run: |
        pip install -r requirements.txt
        pip install comet-ml
    
    - name: Run tests
      run: |
        python -m pytest tests/
    
    - name: Run CometML experiment
      env:
        COMET_API_KEY: ${{ secrets.COMET_API_KEY }}
      run: |
        python run_experiment.py
    
    - name: Deploy model
      if: github.ref == 'refs/heads/main'
      env:
        COMET_API_KEY: ${{ secrets.COMET_API_KEY }}
      run: |
        python deploy_model.py
```

### 2. **Model Deployment Pipeline**
```python
# model_deployment_pipeline.py
import comet_ml
import subprocess
import time
from typing import Dict, Any

def deploy_model_pipeline(model_name: str, version: str, environment: str):
    """Deploy model through staging to production"""
    
    api = comet_ml.API()
    
    # 1. Get model from registry
    model = api.get_model(
        workspace="my-workspace",
        model_name=model_name
    )
    
    # 2. Deploy to staging
    print("Deploying to staging...")
    staging_deployment = model.deploy(
        platform="sagemaker",
        endpoint_name=f"{model_name}-staging",
        instance_type="ml.m5.large"
    )
    
    # 3. Run integration tests
    print("Running integration tests...")
    test_result = run_integration_tests(staging_deployment.endpoint_url)
    
    if test_result.success:
        # 4. Deploy to production
        print("Deploying to production...")
        production_deployment = model.deploy(
            platform="sagemaker",
            endpoint_name=f"{model_name}-production",
            instance_type="ml.m5.xlarge"
        )
        
        # 5. Update model registry
        model.add_version(
            version=version,
            stage="production",
            metadata={
                "deployment_date": datetime.now().isoformat(),
                "environment": environment,
                "endpoint_url": production_deployment.endpoint_url
            }
        )
        
        return production_deployment
    else:
        print("Integration tests failed. Deployment aborted.")
        return None
```

## 📈 Model Monitoring Best Practices

### 1. **Model Performance Monitoring**
```python
# model_monitoring.py
import comet_ml
import pandas as pd
from sklearn.metrics import accuracy_score, precision_score, recall_score
import time
from datetime import datetime

def monitor_model_performance(
    model_id: str,
    test_data: pd.DataFrame,
    y_true: pd.Series
):
    """Monitor model performance in production"""
    
    monitor = comet_ml.ModelMonitor(model_id=model_id)
    
    # Load model (in real scenario, this would be from your serving endpoint)
    model = load_production_model(model_id)
    
    # Make predictions
    predictions = model.predict(test_data)
    
    # Calculate metrics
    accuracy = accuracy_score(y_true, predictions)
    precision = precision_score(y_true, predictions, average='weighted')
    recall = recall_score(y_true, predictions, average='weighted')
    
    # Log predictions
    monitor.log_predictions(
        inputs=test_data.values.tolist(),
        outputs=predictions.tolist(),
        metadata={
            "timestamp": datetime.now().isoformat(),
            "batch_size": len(test_data)
        }
    )
    
    # Log metrics
    monitor.log_metrics({
        "accuracy": accuracy,
        "precision": precision,
        "recall": recall,
        "timestamp": time.time()
    })
    
    # Check for performance degradation
    if accuracy < 0.8:  # Threshold
        send_alert(f"Model accuracy dropped to {accuracy}")
    
    return {
        "accuracy": accuracy,
        "precision": precision,
        "recall": recall
    }
```

### 2. **Data Drift Detection**
```python
# data_drift_detection.py
import comet_ml
import pandas as pd
from alibi_detect import TabularDrift
import numpy as np

def detect_data_drift(
    reference_data: pd.DataFrame,
    new_data: pd.DataFrame,
    threshold: float = 0.1
):
    """Detect data drift and log results to CometML"""
    
    experiment = comet_ml.Experiment()
    
    # Initialize drift detector
    drift_detector = TabularDrift(
        reference_data.values,
        p_val=0.05
    )
    
    # Detect drift
    drift_score = drift_detector.score(new_data.values)
    
    # Log drift results
    experiment.log_other("drift_score", drift_score)
    experiment.log_other("drift_threshold", threshold)
    experiment.log_other("drift_detected", str(drift_score > threshold))
    
    # Log data statistics
    experiment.log_other("reference_data_stats", reference_data.describe().to_dict())
    experiment.log_other("new_data_stats", new_data.describe().to_dict())
    
    if drift_score > threshold:
        send_alert(f"Data drift detected! Score: {drift_score}")
        return True
    
    return False
```

### 3. **Model Serving Monitoring**
```python
# model_serving_monitoring.py
import comet_ml
import requests
import time
from datetime import datetime

def monitor_model_serving(
    model_endpoint: str,
    test_data: dict,
    expected_response_time: float = 1.0
):
    """Monitor model serving performance"""
    
    experiment = comet_ml.Experiment()
    
    start_time = time.time()
    
    try:
        # Make request to model endpoint
        response = requests.post(
            model_endpoint,
            json=test_data,
            timeout=10
        )
        
        response_time = time.time() - start_time
        
        # Log monitoring results
        experiment.log_other("response_time", response_time)
        experiment.log_other("status_code", response.status_code)
        experiment.log_other("endpoint", model_endpoint)
        experiment.log_other("monitoring_time", datetime.now().isoformat())
        
        if response.status_code == 200:
            experiment.log_other("endpoint_status", "healthy")
        else:
            experiment.log_other("endpoint_status", "unhealthy")
        
        # Check for performance issues
        if response_time > expected_response_time:
            send_alert(f"Model response time exceeded threshold: {response_time}s")
        
        if response.status_code != 200:
            send_alert(f"Model endpoint returned error: {response.status_code}")
        
        return {
            "response_time": response_time,
            "status_code": response.status_code,
            "healthy": response.status_code == 200
        }
        
    except Exception as e:
        # Log error
        experiment.log_other("endpoint_status", "error")
        experiment.log_other("error_message", str(e))
        
        send_alert(f"Model endpoint error: {str(e)}")
        return {"error": str(e)}
```

## ⚡ Performance Optimization

### 1. **Efficient Logging**
```python
# Efficient logging practices
import comet_ml
import numpy as np
from typing import Dict, Any

def log_metrics_efficiently(metrics: Dict[str, float]):
    """Log multiple metrics efficiently"""
    
    experiment = comet_ml.Experiment()
    
    # Batch log metrics
    experiment.log_metrics(metrics)
    
    # Log large arrays as artifacts
    large_array = np.random.rand(10000, 100)
    np.save("large_array.npy", large_array)
    experiment.log_asset("large_array.npy")
    
    # Use log_other for structured data
    structured_data = {
        "config": {"param1": "value1", "param2": "value2"},
        "results": {"metric1": 0.95, "metric2": 0.87}
    }
    experiment.log_other("structured_data", structured_data)
```

### 2. **Model Optimization**
```python
# Model optimization for serving
import comet_ml
import joblib
from sklearn.ensemble import RandomForestClassifier

def optimize_model_for_serving(model: RandomForestClassifier):
    """Optimize model for production serving"""
    
    experiment = comet_ml.Experiment()
    
    # Reduce model size
    model.n_estimators = min(model.n_estimators, 100)
    
    # Serialize efficiently
    model_path = "optimized_model.pkl"
    joblib.dump(model, model_path, compress=3)
    
    # Log optimized model
    experiment.log_model(model, "optimized_model")
    
    return model_path
```

### 3. **Caching Strategy**
```python
# Implement caching for model serving
import comet_ml
import redis
import pickle
from typing import Any

def cache_model_predictions(
    model_id: str,
    input_data: Any,
    cache_ttl: int = 3600
):
    """Cache model predictions for performance"""
    
    # Initialize Redis cache
    cache = redis.Redis(host='localhost', port=6379, db=0)
    
    # Create cache key
    cache_key = f"model:{model_id}:{hash(str(input_data))}"
    
    # Check cache
    cached_result = cache.get(cache_key)
    if cached_result:
        return pickle.loads(cached_result)
    
    # Load model and make prediction
    model = load_model(model_id)
    prediction = model.predict(input_data)
    
    # Cache result
    cache.setex(cache_key, cache_ttl, pickle.dumps(prediction))
    
    return prediction
```

## 🔧 Reproducibility Best Practices

### 1. **Environment Management**
```yaml
# requirements.txt with exact versions
comet-ml==3.31.0
scikit-learn==1.0.2
pandas==1.3.5
numpy==1.21.6
matplotlib==3.5.1
seaborn==0.11.2
```

### 2. **Random Seed Management**
```python
# Centralized seed management
import random
import numpy as np
import comet_ml

def set_seeds(seed: int = 42):
    """Set random seeds for reproducibility"""
    random.seed(seed)
    np.random.seed(seed)
    
    # Log seed
    experiment = comet_ml.Experiment()
    experiment.log_other("random_seed", seed)
```

### 3. **Parameter Management**
```python
# Centralized parameter management
import comet_ml
import yaml
from typing import Dict, Any

def load_and_log_config(config_path: str) -> Dict[str, Any]:
    """Load configuration and log to CometML"""
    
    experiment = comet_ml.Experiment()
    
    with open(config_path, 'r') as f:
        config = yaml.safe_load(f)
    
    # Log all parameters
    for key, value in config.items():
        if isinstance(value, (int, float, str, bool)):
            experiment.log_parameter(key, value)
        else:
            experiment.log_other(key, value)
    
    return config
```

## 🧪 Testing Best Practices

### 1. **Unit Tests**
```python
# test_cometml_integration.py
import pytest
import comet_ml

def test_cometml_tracking():
    """Test CometML tracking functionality"""
    
    # Initialize experiment
    experiment = comet_ml.Experiment(
        project_name="test-project",
        offline=True
    )
    
    # Test parameter logging
    experiment.log_parameter("test_param", "test_value")
    
    # Test metric logging
    experiment.log_metric("test_metric", 0.95)
    
    # Test model logging
    from sklearn.ensemble import RandomForestClassifier
    model = RandomForestClassifier(n_estimators=10, random_state=42)
    experiment.log_model(model, "test_model")
    
    # End experiment
    experiment.end()
    
    # Verify experiment data
    assert experiment.get_parameter("test_param") == "test_value"
    assert experiment.get_metric("test_metric") == 0.95
```

### 2. **Integration Tests**
```python
# test_model_serving.py
import pytest
import requests
import comet_ml

def test_model_serving():
    """Test model serving functionality"""
    
    # Deploy model
    api = comet_ml.API()
    model = api.get_model("test-model")
    
    deployment = model.deploy(
        platform="sagemaker",
        endpoint_name="test-endpoint"
    )
    
    # Test model serving
    test_data = {"data": [[1, 2, 3, 4]]}
    response = requests.post(
        deployment.endpoint_url,
        json=test_data
    )
    
    assert response.status_code == 200
    assert "predictions" in response.json()
```

## 📚 Documentation Best Practices

### 1. **Experiment Documentation**
```python
# Document experiments with CometML
import comet_ml
from typing import Dict, Any

def document_experiment(
    experiment_name: str,
    description: str,
    tags: Dict[str, str]
):
    """Document experiment with comprehensive metadata"""
    
    experiment = comet_ml.Experiment(
        experiment_name=experiment_name,
        tags=list(tags.keys())
    )
    
    # Log experiment description
    experiment.log_other("description", description)
    experiment.log_other("author", "data-scientist")
    experiment.log_other("date", datetime.now().isoformat())
    
    # Log experiment tags
    for key, value in tags.items():
        experiment.log_other(f"tag_{key}", value)
```

### 2. **Model Documentation**
```python
# Document models with CometML
import comet_ml

def document_model(
    model_name: str,
    version: str,
    description: str,
    performance_metrics: Dict[str, float]
):
    """Document model with comprehensive metadata"""
    
    api = comet_ml.API()
    model = api.get_model(
        workspace="my-workspace",
        model_name=model_name
    )
    
    # Add model version with metadata
    model.add_version(
        version=version,
        metadata={
            "description": description,
            "performance_metrics": performance_metrics,
            "documentation_date": datetime.now().isoformat()
        }
    )
```

## 🚨 Common Pitfalls to Avoid

### 1. **Don't Log Sensitive Data**
```python
# Bad: Logging sensitive data
experiment.log_parameter("api_key", "secret-key")
experiment.log_parameter("password", "secret-password")

# Good: Sanitize sensitive data
experiment.log_parameter("api_key", "[REDACTED]")
experiment.log_parameter("password", "[REDACTED]")
```

### 2. **Don't Ignore Model Versioning**
```python
# Bad: No model versioning
experiment.log_model(model, "model")

# Good: Proper model versioning
api = comet_ml.API()
model = api.get_model("production-model")
model.add_version("1.0.0", path="model.pkl")
```

### 3. **Don't Hardcode Configuration**
```python
# Bad: Hardcoded configuration
experiment = comet_ml.Experiment(api_key="hardcoded-key")

# Good: Configurable API key
import os
experiment = comet_ml.Experiment(api_key=os.getenv("COMET_API_KEY"))
```

---

*Follow these best practices to build robust, scalable ML systems with CometML! 🎯*