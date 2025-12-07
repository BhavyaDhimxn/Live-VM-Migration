# WandB — Best Practices

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
import wandb
from datetime import datetime

def create_experiment(experiment_type: str, model_name: str, version: str):
    """Create experiment with consistent naming"""
    
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    experiment_name = f"{experiment_type}_{model_name}_v{version}_{timestamp}"
    
    wandb.init(
        project="ml-project",
        name=experiment_name,
        tags=[experiment_type, model_name, version]
    )
    
    return wandb.run

# Usage
run = create_experiment("training", "random-forest", "1.0")
```

## 🔐 Security Best Practices

### 1. **API Key Management**
```python
# Secure API key handling
import os
import wandb

def get_api_key():
    """Get API key securely"""
    
    # Try environment variable first
    api_key = os.getenv("WANDB_API_KEY")
    
    if not api_key:
        # Try wandb login
        wandb.login()
        api_key = os.getenv("WANDB_API_KEY")
    
    if not api_key:
        raise ValueError("API key is required")
    
    return api_key

# Initialize wandb with secure API key
wandb.login(key=get_api_key())
```

### 2. **Data Privacy Controls**
```python
# Data privacy and sanitization
import wandb
import pandas as pd

class PrivacyAwareWandB:
    def __init__(self, project_name: str):
        self.project_name = project_name
        self.sensitive_fields = ['ssn', 'credit_card', 'email', 'phone']
        wandb.init(project=project_name)
    
    def log_data_safely(self, data: pd.DataFrame, data_name: str):
        """Log data with privacy controls"""
        
        # Sanitize sensitive data
        sanitized_data = self._sanitize_data(data)
        
        # Log data statistics instead of raw data
        wandb.log({
            f"{data_name}_shape": data.shape,
            f"{data_name}_columns": list(data.columns),
            f"{data_name}_dtypes": data.dtypes.to_dict()
        })
        
        # Log summary statistics
        summary_stats = data.describe().to_dict()
        wandb.log({f"{data_name}_summary": summary_stats})
        
        # Log sample data (first 100 rows) as table
        sample_data = sanitized_data.head(100)
        wandb.log({f"{data_name}_sample": wandb.Table(dataframe=sample_data)})
    
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
import wandb

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
    
    wandb.init(
        project=project_name,
        tags=[f"role:{user_role}", "restricted"]
    )
    
    # Log access information
    wandb.log({
        "user_role": user_role,
        "access_levels": access_levels[user_role]
    })
    
    return wandb.run
```

## 📊 Data Management Best Practices

### 1. **Data Versioning Strategy**
```python
# Implement data versioning with WandB
import wandb
import hashlib
import pandas as pd
from datetime import datetime

def log_data_version(data: pd.DataFrame, data_name: str, data_source: str):
    """Log data version and metadata"""
    
    wandb.init(project="data-versioning")
    
    # Calculate data hash
    data_hash = hashlib.md5(data.to_string().encode()).hexdigest()
    
    # Log data metadata
    wandb.log({
        f"{data_name}_hash": data_hash,
        f"{data_name}_shape": data.shape,
        f"{data_name}_columns": list(data.columns),
        f"{data_name}_source": data_source,
        f"{data_name}_timestamp": datetime.now().isoformat()
    })
    
    # Log data quality metrics
    wandb.log({
        f"{data_name}_null_count": data.isnull().sum().to_dict(),
        f"{data_name}_duplicate_count": data.duplicated().sum()
    })
    
    return data_hash
```

### 2. **Data Quality Validation**
```python
# Implement data quality validation
import wandb
import pandas as pd
from great_expectations import dataset

def validate_data_quality(data: pd.DataFrame, expectations_suite: str):
    """Validate data quality and log results"""
    
    wandb.init(project="data-quality")
    
    # Load expectations
    ge_df = dataset.PandasDataset(data)
    
    # Run validations
    validation_result = ge_df.validate(expectation_suite=expectations_suite)
    
    # Log validation results
    wandb.log({
        "data_quality_score": validation_result.success_rate,
        "data_validation_result": validation_result.to_json_dict()
    })
    
    if not validation_result.success:
        wandb.log({
            "data_quality_issues": "true",
            "failed_expectations": validation_result.failed_expectations
        })
        raise ValueError("Data quality validation failed")
    
    return validation_result
```

### 3. **Data Lineage Tracking**
```python
# Track data lineage through WandB
import wandb
from typing import Dict, List
from datetime import datetime

def track_data_lineage(
    input_data: Dict[str, str],
    transformations: List[str],
    output_data: Dict[str, str]
):
    """Track data lineage through WandB"""
    
    wandb.init(project="data-lineage")
    
    # Log input data sources
    for name, path in input_data.items():
        wandb.log({f"input_data_{name}": path})
    
    # Log transformations
    wandb.log({"transformations": transformations})
    
    # Log output data
    for name, path in output_data.items():
        wandb.log({f"output_data_{name}": path})
    
    # Log lineage graph
    lineage = {
        "inputs": input_data,
        "transformations": transformations,
        "outputs": output_data,
        "timestamp": datetime.now().isoformat()
    }
    wandb.log({"data_lineage": lineage})
```

## 🔄 CI/CD Integration

### 1. **Automated Experiment Tracking**
```yaml
# .github/workflows/wandb-ci-cd.yml
name: WandB CI/CD

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
        pip install wandb
    
    - name: Run tests
      run: |
        python -m pytest tests/
    
    - name: Run WandB experiment
      env:
        WANDB_API_KEY: ${{ secrets.WANDB_API_KEY }}
      run: |
        python run_experiment.py
    
    - name: Deploy model
      if: github.ref == 'refs/heads/main'
      env:
        WANDB_API_KEY: ${{ secrets.WANDB_API_KEY }}
      run: |
        python deploy_model.py
```

### 2. **Model Deployment Pipeline**
```python
# model_deployment_pipeline.py
import wandb
import subprocess
import time
from typing import Dict, Any
from datetime import datetime

def deploy_model_pipeline(model_name: str, version: str, environment: str):
    """Deploy model through staging to production"""
    
    api = wandb.Api()
    
    # 1. Get model from registry
    model = api.artifact(f"{model_name}:{version}")
    
    # 2. Deploy to staging
    print("Deploying to staging...")
    staging_deployment = deploy_to_staging(model)
    
    # 3. Run integration tests
    print("Running integration tests...")
    test_result = run_integration_tests(staging_deployment.endpoint_url)
    
    if test_result.success:
        # 4. Deploy to production
        print("Deploying to production...")
        production_deployment = deploy_to_production(model)
        
        # 5. Update model registry
        model.aliases.append("production")
        model.save()
        
        return production_deployment
    else:
        print("Integration tests failed. Deployment aborted.")
        return None
```

## 📈 Model Monitoring Best Practices

### 1. **Model Performance Monitoring**
```python
# model_monitoring.py
import wandb
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
    
    wandb.init(
        project="model-monitoring",
        name=f"monitoring-{model_id}"
    )
    
    # Load model (in real scenario, this would be from your serving endpoint)
    model = load_production_model(model_id)
    
    # Make predictions
    predictions = model.predict(test_data)
    
    # Calculate metrics
    accuracy = accuracy_score(y_true, predictions)
    precision = precision_score(y_true, predictions, average='weighted')
    recall = recall_score(y_true, predictions, average='weighted')
    
    # Log predictions
    wandb.log({
        "predictions": predictions.tolist(),
        "ground_truth": y_true.tolist(),
        "timestamp": time.time()
    })
    
    # Log metrics
    wandb.log({
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
import wandb
import pandas as pd
from alibi_detect import TabularDrift
import numpy as np

def detect_data_drift(
    reference_data: pd.DataFrame,
    new_data: pd.DataFrame,
    threshold: float = 0.1
):
    """Detect data drift and log results to WandB"""
    
    wandb.init(project="data-drift-detection")
    
    # Initialize drift detector
    drift_detector = TabularDrift(
        reference_data.values,
        p_val=0.05
    )
    
    # Detect drift
    drift_score = drift_detector.score(new_data.values)
    
    # Log drift results
    wandb.log({
        "drift_score": drift_score,
        "drift_threshold": threshold,
        "drift_detected": str(drift_score > threshold)
    })
    
    # Log data statistics
    wandb.log({
        "reference_data_stats": reference_data.describe().to_dict(),
        "new_data_stats": new_data.describe().to_dict()
    })
    
    if drift_score > threshold:
        send_alert(f"Data drift detected! Score: {drift_score}")
        return True
    
    return False
```

### 3. **Model Serving Monitoring**
```python
# model_serving_monitoring.py
import wandb
import requests
import time
from datetime import datetime

def monitor_model_serving(
    model_endpoint: str,
    test_data: dict,
    expected_response_time: float = 1.0
):
    """Monitor model serving performance"""
    
    wandb.init(project="model-serving-monitoring")
    
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
        wandb.log({
            "response_time": response_time,
            "status_code": response.status_code,
            "endpoint": model_endpoint,
            "monitoring_time": datetime.now().isoformat()
        })
        
        if response.status_code == 200:
            wandb.log({"endpoint_status": "healthy"})
        else:
            wandb.log({"endpoint_status": "unhealthy"})
        
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
        wandb.log({
            "endpoint_status": "error",
            "error_message": str(e)
        })
        
        send_alert(f"Model endpoint error: {str(e)}")
        return {"error": str(e)}
```

## ⚡ Performance Optimization

### 1. **Efficient Logging**
```python
# Efficient logging practices
import wandb
import numpy as np
from typing import Dict, Any

def log_metrics_efficiently(metrics: Dict[str, float]):
    """Log multiple metrics efficiently"""
    
    wandb.init(project="efficient-logging")
    
    # Batch log metrics
    wandb.log(metrics)
    
    # Log large arrays as artifacts
    large_array = np.random.rand(10000, 100)
    np.save("large_array.npy", large_array)
    wandb.log_artifact("large_array.npy", type="dataset")
    
    # Use wandb.Table for structured data
    structured_data = wandb.Table(
        columns=["param1", "param2", "metric1", "metric2"],
        data=[["value1", "value2", 0.95, 0.87]]
    )
    wandb.log({"structured_data": structured_data})
```

### 2. **Model Optimization**
```python
# Model optimization for serving
import wandb
import joblib
from sklearn.ensemble import RandomForestClassifier

def optimize_model_for_serving(model: RandomForestClassifier):
    """Optimize model for production serving"""
    
    wandb.init(project="model-optimization")
    
    # Reduce model size
    model.n_estimators = min(model.n_estimators, 100)
    
    # Serialize efficiently
    model_path = "optimized_model.pkl"
    joblib.dump(model, model_path, compress=3)
    
    # Log optimized model
    wandb.log_artifact(model_path, type="model")
    
    return model_path
```

### 3. **Caching Strategy**
```python
# Implement caching for model serving
import wandb
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
```txt
# requirements.txt with exact versions
wandb==0.15.0
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
import wandb

def set_seeds(seed: int = 42):
    """Set random seeds for reproducibility"""
    random.seed(seed)
    np.random.seed(seed)
    
    # Log seed
    wandb.config.update({"random_seed": seed})
```

### 3. **Parameter Management**
```python
# Centralized parameter management
import wandb
import yaml
from typing import Dict, Any

def load_and_log_config(config_path: str) -> Dict[str, Any]:
    """Load configuration and log to WandB"""
    
    wandb.init(project="config-management")
    
    with open(config_path, 'r') as f:
        config = yaml.safe_load(f)
    
    # Log all parameters
    wandb.config.update(config)
    
    return config
```

## 🧪 Testing Best Practices

### 1. **Unit Tests**
```python
# test_wandb_integration.py
import pytest
import wandb

def test_wandb_tracking():
    """Test WandB tracking functionality"""
    
    # Initialize wandb in offline mode
    wandb.init(project="test-project", mode="offline")
    
    # Test parameter logging
    wandb.config.update({"test_param": "test_value"})
    
    # Test metric logging
    wandb.log({"test_metric": 0.95})
    
    # Test model logging
    from sklearn.ensemble import RandomForestClassifier
    model = RandomForestClassifier(n_estimators=10, random_state=42)
    wandb.log_model(model, "test_model")
    
    # Finish run
    wandb.finish()
    
    # Verify wandb data
    assert wandb.run.config.test_param == "test_value"
```

### 2. **Integration Tests**
```python
# test_model_serving.py
import pytest
import requests
import wandb

def test_model_serving():
    """Test model serving functionality"""
    
    # Get model from registry
    api = wandb.Api()
    model = api.artifact("test-model:latest")
    
    # Download and load model
    model_dir = model.download()
    # Load model and test serving
    # (Implementation depends on serving platform)
    
    # Test model predictions
    test_data = {"data": [[1, 2, 3, 4]]}
    # Make prediction request
    # Assert response
```

## 📚 Documentation Best Practices

### 1. **Experiment Documentation**
```python
# Document experiments with WandB
import wandb
from typing import Dict, Any

def document_experiment(
    experiment_name: str,
    description: str,
    tags: Dict[str, str]
):
    """Document experiment with comprehensive metadata"""
    
    wandb.init(
        name=experiment_name,
        tags=list(tags.keys()),
        notes=description
    )
    
    # Log experiment metadata
    wandb.log({
        "description": description,
        "author": "data-scientist",
        "date": datetime.now().isoformat()
    })
    
    # Log experiment tags
    for key, value in tags.items():
        wandb.log({f"tag_{key}": value})
```

### 2. **Model Documentation**
```python
# Document models with WandB
import wandb

def document_model(
    model_name: str,
    version: str,
    description: str,
    performance_metrics: Dict[str, float]
):
    """Document model with comprehensive metadata"""
    
    api = wandb.Api()
    model = api.artifact(f"{model_name}:{version}")
    
    # Add model metadata
    model.metadata.update({
        "description": description,
        "performance_metrics": performance_metrics,
        "documentation_date": datetime.now().isoformat()
    })
    
    model.save()
```

## 🚨 Common Pitfalls to Avoid

### 1. **Don't Log Sensitive Data**
```python
# Bad: Logging sensitive data
wandb.config.update({"api_key": "secret-key"})
wandb.config.update({"password": "secret-password"})

# Good: Sanitize sensitive data
wandb.config.update({"api_key": "[REDACTED]"})
wandb.config.update({"password": "[REDACTED]"})
```

### 2. **Don't Ignore Model Versioning**
```python
# Bad: No model versioning
wandb.log_model(model, "model")

# Good: Proper model versioning
wandb.log_model(
    model,
    "production-model",
    aliases=["latest", "production", "v1.0.0"]
)
```

### 3. **Don't Hardcode Configuration**
```python
# Bad: Hardcoded configuration
wandb.init(project="hardcoded-project")

# Good: Configurable project
import os
wandb.init(project=os.getenv("WANDB_PROJECT", "default-project"))
```

---

*Follow these best practices to build robust, scalable ML systems with WandB! 🎯*