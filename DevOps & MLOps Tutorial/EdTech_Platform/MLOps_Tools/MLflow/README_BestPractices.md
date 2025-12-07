# MLflow — Best Practices

## 🎯 Production Best Practices

### 1. **Project Structure**
```
ml-project/
├── MLproject
├── conda.yaml
├── requirements.txt
├── src/
│   ├── __init__.py
│   ├── data_preparation.py
│   ├── model_training.py
│   └── model_evaluation.py
├── data/
│   ├── raw/
│   ├── processed/
│   └── external/
├── models/
├── notebooks/
├── tests/
└── configs/
    ├── base.yaml
    ├── local.yaml
    └── production.yaml
```

### 2. **Experiment Organization**
```python
# Organize experiments by project and phase
import mlflow

# Create experiment hierarchy
mlflow.create_experiment("project-name/data-preparation")
mlflow.create_experiment("project-name/model-training")
mlflow.create_experiment("project-name/model-evaluation")
mlflow.create_experiment("project-name/hyperparameter-tuning")

# Set experiment context
mlflow.set_experiment("project-name/model-training")
```

## 🔐 Security Best Practices

### 1. **Authentication and Authorization**
```bash
# Enable authentication
mlflow server --app-name basic-auth --backend-store-uri postgresql://user:password@localhost:5432/mlflow

# Use environment variables for credentials
export MLFLOW_TRACKING_URI="http://username:password@localhost:5000"
```

### 2. **Secure Model Registry**
```python
# Implement model approval workflow
from mlflow.tracking import MlflowClient

client = MlflowClient()

def approve_model(model_name: str, version: int, approver: str):
    """Approve model for production deployment"""
    
    # Add approval metadata
    client.set_model_version_tag(
        name=model_name,
        version=version,
        key="approved_by",
        value=approver
    )
    
    client.set_model_version_tag(
        name=model_name,
        version=version,
        key="approval_date",
        value=str(datetime.now())
    )
    
    # Transition to production
    client.transition_model_version_stage(
        name=model_name,
        version=version,
        stage="Production"
    )
```

### 3. **Data Privacy**
```python
# Implement data privacy controls
import mlflow
from mlflow.tracking import MlflowClient

class PrivacyAwareMLflow:
    def __init__(self, tracking_uri: str):
        self.client = MlflowClient(tracking_uri)
        
    def log_sensitive_data(self, data: dict, run_id: str):
        """Log data with privacy controls"""
        
        # Remove sensitive fields
        sanitized_data = self._sanitize_data(data)
        
        # Log sanitized data
        mlflow.log_dict(sanitized_data, "data.json")
        
    def _sanitize_data(self, data: dict) -> dict:
        """Remove sensitive information from data"""
        sensitive_fields = ['ssn', 'credit_card', 'email', 'phone']
        
        sanitized = data.copy()
        for field in sensitive_fields:
            if field in sanitized:
                sanitized[field] = "[REDACTED]"
        
        return sanitized
```

## 📊 Data Management Best Practices

### 1. **Data Versioning Strategy**
```python
# Implement data versioning with MLflow
import mlflow
import hashlib
import pandas as pd

def log_data_version(data: pd.DataFrame, data_name: str):
    """Log data version and hash"""
    
    # Calculate data hash
    data_hash = hashlib.md5(data.to_string().encode()).hexdigest()
    
    # Log data metadata
    mlflow.log_param(f"{data_name}_hash", data_hash)
    mlflow.log_param(f"{data_name}_shape", data.shape)
    mlflow.log_param(f"{data_name}_columns", list(data.columns))
    
    # Log data sample
    mlflow.log_table(data.head(100), f"{data_name}_sample.json")
    
    return data_hash
```

### 2. **Data Quality Checks**
```python
# Implement data quality validation
import mlflow
import pandas as pd
from great_expectations import dataset

def validate_data_quality(data: pd.DataFrame, expectations_suite: str):
    """Validate data quality and log results"""
    
    # Load expectations
    ge_df = dataset.PandasDataset(data)
    
    # Run validations
    validation_result = ge_df.validate(expectation_suite=expectations_suite)
    
    # Log validation results
    mlflow.log_metric("data_quality_score", validation_result.success_rate)
    mlflow.log_dict(validation_result.to_json_dict(), "data_validation.json")
    
    if not validation_result.success:
        mlflow.log_param("data_quality_issues", "true")
        raise ValueError("Data quality validation failed")
    
    return validation_result
```

### 3. **Data Lineage Tracking**
```python
# Track data lineage through MLflow
import mlflow
from typing import Dict, List

def track_data_lineage(
    input_data: Dict[str, str],
    transformations: List[str],
    output_data: Dict[str, str]
):
    """Track data lineage through MLflow"""
    
    # Log input data sources
    for name, path in input_data.items():
        mlflow.log_param(f"input_data_{name}", path)
    
    # Log transformations
    mlflow.log_param("transformations", ",".join(transformations))
    
    # Log output data
    for name, path in output_data.items():
        mlflow.log_param(f"output_data_{name}", path)
    
    # Log lineage graph
    lineage = {
        "inputs": input_data,
        "transformations": transformations,
        "outputs": output_data
    }
    mlflow.log_dict(lineage, "data_lineage.json")
```

## 🔄 CI/CD Integration

### 1. **Automated Model Training**
```yaml
# .github/workflows/mlflow-ci-cd.yml
name: MLflow CI/CD

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
        pip install mlflow
    
    - name: Run tests
      run: |
        python -m pytest tests/
    
    - name: Run MLflow project
      run: |
        mlflow run . --experiment-name "ci-experiment"
    
    - name: Deploy to staging
      if: github.ref == 'refs/heads/main'
      run: |
        mlflow models serve -m "models:/staging-model/1" -p 5000
```

### 2. **Model Deployment Pipeline**
```python
# model_deployment_pipeline.py
import mlflow
from mlflow.tracking import MlflowClient
import subprocess
import time

def deploy_model_pipeline(model_name: str, version: int):
    """Deploy model through staging to production"""
    
    client = MlflowClient()
    
    # 1. Deploy to staging
    print("Deploying to staging...")
    staging_process = subprocess.Popen([
        "mlflow", "models", "serve",
        "-m", f"models:/{model_name}/{version}",
        "-p", "5000",
        "--env-manager", "conda"
    ])
    
    # Wait for staging deployment
    time.sleep(30)
    
    # 2. Run integration tests
    print("Running integration tests...")
    test_result = run_integration_tests("http://localhost:5000")
    
    if test_result.success:
        # 3. Deploy to production
        print("Deploying to production...")
        client.transition_model_version_stage(
            name=model_name,
            version=version,
            stage="Production"
        )
        
        # 4. Update production endpoint
        update_production_endpoint(model_name, version)
    else:
        print("Integration tests failed. Deployment aborted.")
        staging_process.terminate()
        
    return test_result
```

## 📈 Model Monitoring Best Practices

### 1. **Model Performance Monitoring**
```python
# model_monitoring.py
import mlflow
import pandas as pd
from sklearn.metrics import accuracy_score, precision_score, recall_score
import time
from datetime import datetime

def monitor_model_performance(
    model_uri: str,
    test_data: pd.DataFrame,
    y_true: pd.Series
):
    """Monitor model performance in production"""
    
    # Load model
    model = mlflow.pyfunc.load_model(model_uri)
    
    # Make predictions
    predictions = model.predict(test_data)
    
    # Calculate metrics
    accuracy = accuracy_score(y_true, predictions)
    precision = precision_score(y_true, predictions, average='weighted')
    recall = recall_score(y_true, predictions, average='weighted')
    
    # Log metrics to MLflow
    with mlflow.start_run() as run:
        mlflow.log_metric("accuracy", accuracy)
        mlflow.log_metric("precision", precision)
        mlflow.log_metric("recall", recall)
        mlflow.log_metric("timestamp", time.time())
        mlflow.log_param("model_uri", model_uri)
        mlflow.log_param("monitoring_date", datetime.now().isoformat())
    
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
import mlflow
import pandas as pd
from alibi_detect import TabularDrift
import numpy as np

def detect_data_drift(
    reference_data: pd.DataFrame,
    new_data: pd.DataFrame,
    threshold: float = 0.1
):
    """Detect data drift and log results to MLflow"""
    
    # Initialize drift detector
    drift_detector = TabularDrift(
        reference_data.values,
        p_val=0.05
    )
    
    # Detect drift
    drift_score = drift_detector.score(new_data.values)
    
    # Log drift results
    with mlflow.start_run() as run:
        mlflow.log_metric("drift_score", drift_score)
        mlflow.log_param("drift_threshold", threshold)
        mlflow.log_param("drift_detected", str(drift_score > threshold))
        
        # Log data statistics
        mlflow.log_dict({
            "reference_data_shape": reference_data.shape,
            "new_data_shape": new_data.shape,
            "reference_data_stats": reference_data.describe().to_dict(),
            "new_data_stats": new_data.describe().to_dict()
        }, "data_statistics.json")
    
    if drift_score > threshold:
        send_alert(f"Data drift detected! Score: {drift_score}")
        return True
    
    return False
```

### 3. **Model Serving Monitoring**
```python
# model_serving_monitoring.py
import mlflow
import requests
import time
from datetime import datetime

def monitor_model_serving(
    model_endpoint: str,
    test_data: dict,
    expected_response_time: float = 1.0
):
    """Monitor model serving performance"""
    
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
        with mlflow.start_run() as run:
            mlflow.log_metric("response_time", response_time)
            mlflow.log_metric("status_code", response.status_code)
            mlflow.log_param("endpoint", model_endpoint)
            mlflow.log_param("monitoring_time", datetime.now().isoformat())
            
            if response.status_code == 200:
                mlflow.log_param("endpoint_status", "healthy")
            else:
                mlflow.log_param("endpoint_status", "unhealthy")
        
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
        with mlflow.start_run() as run:
            mlflow.log_param("endpoint_status", "error")
            mlflow.log_param("error_message", str(e))
        
        send_alert(f"Model endpoint error: {str(e)}")
        return {"error": str(e)}
```

## ⚡ Performance Optimization

### 1. **Efficient Logging**
```python
# Efficient logging practices
import mlflow
import numpy as np
from typing import Dict, Any

def log_metrics_efficiently(metrics: Dict[str, float]):
    """Log multiple metrics efficiently"""
    
    # Batch log metrics
    mlflow.log_metrics(metrics)
    
    # Log large arrays as artifacts
    large_array = np.random.rand(10000, 100)
    np.save("large_array.npy", large_array)
    mlflow.log_artifact("large_array.npy")
    
    # Use log_dict for structured data
    structured_data = {
        "config": {"param1": "value1", "param2": "value2"},
        "results": {"metric1": 0.95, "metric2": 0.87}
    }
    mlflow.log_dict(structured_data, "results.json")
```

### 2. **Model Optimization**
```python
# Model optimization for serving
import mlflow
import mlflow.pyfunc
from sklearn.ensemble import RandomForestClassifier
import joblib

def optimize_model_for_serving(model: RandomForestClassifier):
    """Optimize model for production serving"""
    
    # Reduce model size
    model.n_estimators = min(model.n_estimators, 100)
    
    # Serialize efficiently
    model_path = "optimized_model.pkl"
    joblib.dump(model, model_path, compress=3)
    
    # Log optimized model
    mlflow.log_artifact(model_path, "optimized_model")
    
    return model_path
```

### 3. **Caching Strategy**
```python
# Implement caching for model serving
import mlflow
import redis
import pickle
from typing import Any

def cache_model_predictions(
    model_uri: str,
    input_data: Any,
    cache_ttl: int = 3600
):
    """Cache model predictions for performance"""
    
    # Initialize Redis cache
    cache = redis.Redis(host='localhost', port=6379, db=0)
    
    # Create cache key
    cache_key = f"model:{model_uri}:{hash(str(input_data))}"
    
    # Check cache
    cached_result = cache.get(cache_key)
    if cached_result:
        return pickle.loads(cached_result)
    
    # Load model and make prediction
    model = mlflow.pyfunc.load_model(model_uri)
    prediction = model.predict(input_data)
    
    # Cache result
    cache.setex(cache_key, cache_ttl, pickle.dumps(prediction))
    
    return prediction
```

## 🔧 Reproducibility Best Practices

### 1. **Environment Management**
```yaml
# conda.yaml with exact versions
name: mlflow-project
channels:
  - conda-forge
dependencies:
  - python=3.9.7
  - scikit-learn=1.0.2
  - pandas=1.3.5
  - numpy=1.21.6
  - matplotlib=3.5.1
  - seaborn=0.11.2
  - pip
  - pip:
    - mlflow==2.1.1
    - great-expectations==0.15.0
```

### 2. **Random Seed Management**
```python
# Centralized seed management
import random
import numpy as np
import torch
import tensorflow as tf

def set_seeds(seed: int = 42):
    """Set random seeds for reproducibility"""
    random.seed(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)
    if torch.cuda.is_available():
        torch.cuda.manual_seed(seed)
        torch.cuda.manual_seed_all(seed)
    tf.random.set_seed(seed)
    
    # Log seed
    mlflow.log_param("random_seed", seed)
```

### 3. **Parameter Management**
```python
# Centralized parameter management
import mlflow
import yaml
from typing import Dict, Any

def load_and_log_config(config_path: str) -> Dict[str, Any]:
    """Load configuration and log to MLflow"""
    
    with open(config_path, 'r') as f:
        config = yaml.safe_load(f)
    
    # Log all parameters
    for key, value in config.items():
        if isinstance(value, (int, float, str, bool)):
            mlflow.log_param(key, value)
        else:
            mlflow.log_dict({key: value}, f"{key}.json")
    
    return config
```

## 🧪 Testing Best Practices

### 1. **Unit Tests**
```python
# test_mlflow_integration.py
import pytest
import mlflow
from mlflow.tracking import MlflowClient

def test_mlflow_tracking():
    """Test MLflow tracking functionality"""
    
    # Set tracking URI
    mlflow.set_tracking_uri("sqlite:///test.db")
    
    # Create test experiment
    experiment_id = mlflow.create_experiment("test-experiment")
    
    # Test run tracking
    with mlflow.start_run(experiment_id=experiment_id) as run:
        mlflow.log_param("test_param", "test_value")
        mlflow.log_metric("test_metric", 0.95)
        
        # Verify run data
        client = MlflowClient()
        run_data = client.get_run(run.info.run_id)
        
        assert run_data.data.params["test_param"] == "test_value"
        assert run_data.data.metrics["test_metric"] == 0.95
```

### 2. **Integration Tests**
```python
# test_model_serving.py
import pytest
import requests
import mlflow
import mlflow.sklearn
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_classification

def test_model_serving():
    """Test model serving functionality"""
    
    # Train and log model
    X, y = make_classification(n_samples=100, n_features=4, random_state=42)
    model = RandomForestClassifier(n_estimators=10, random_state=42)
    model.fit(X, y)
    
    with mlflow.start_run() as run:
        model_uri = mlflow.sklearn.log_model(model, "model").model_uri
    
    # Test model serving
    # (This would require a running MLflow server)
    # response = requests.post(
    #     "http://localhost:5000/invocations",
    #     json={"dataframe_split": {"data": X[:5].tolist()}}
    # )
    # assert response.status_code == 200
```

## 📚 Documentation Best Practices

### 1. **Experiment Documentation**
```python
# Document experiments with MLflow
import mlflow
from typing import Dict, Any

def document_experiment(
    experiment_name: str,
    description: str,
    tags: Dict[str, str]
):
    """Document experiment with comprehensive metadata"""
    
    # Create experiment with description
    experiment_id = mlflow.create_experiment(
        name=experiment_name,
        tags=tags
    )
    
    # Log experiment description
    with mlflow.start_run(experiment_id=experiment_id) as run:
        mlflow.log_param("experiment_description", description)
        mlflow.log_param("experiment_author", "data-scientist")
        mlflow.log_param("experiment_date", datetime.now().isoformat())
        
        # Log experiment tags
        for key, value in tags.items():
            mlflow.log_param(f"tag_{key}", value)
```

### 2. **Model Documentation**
```python
# Document models with MLflow
import mlflow
from mlflow.tracking import MlflowClient

def document_model(
    model_name: str,
    version: int,
    description: str,
    performance_metrics: Dict[str, float]
):
    """Document model with comprehensive metadata"""
    
    client = MlflowClient()
    
    # Add model description
    client.set_model_version_tag(
        name=model_name,
        version=version,
        key="description",
        value=description
    )
    
    # Add performance metrics
    for metric_name, metric_value in performance_metrics.items():
        client.set_model_version_tag(
            name=model_name,
            version=version,
            key=f"metric_{metric_name}",
            value=str(metric_value)
        )
    
    # Add model metadata
    client.set_model_version_tag(
        name=model_name,
        version=version,
        key="documentation_date",
        value=datetime.now().isoformat()
    )
```

## 🚨 Common Pitfalls to Avoid

### 1. **Don't Log Sensitive Data**
```python
# Bad: Logging sensitive data
mlflow.log_param("api_key", "secret-key")
mlflow.log_param("password", "secret-password")

# Good: Sanitize sensitive data
mlflow.log_param("api_key", "[REDACTED]")
mlflow.log_param("password", "[REDACTED]")
```

### 2. **Don't Ignore Model Versioning**
```python
# Bad: No model versioning
mlflow.sklearn.log_model(model, "model")

# Good: Proper model versioning
mlflow.sklearn.log_model(
    model, 
    "model",
    registered_model_name="production-model"
)
```

### 3. **Don't Hardcode Configuration**
```python
# Bad: Hardcoded configuration
mlflow.set_tracking_uri("http://localhost:5000")

# Good: Configurable tracking URI
import os
mlflow.set_tracking_uri(os.getenv("MLFLOW_TRACKING_URI", "http://localhost:5000"))
```

---

*Follow these best practices to build robust, scalable ML systems with MLflow! 🎯*