# Kubeflow — Best Practices

## 🎯 Production Best Practices

### 1. **Cluster Architecture**
```
Production Kubeflow Cluster:
├── Control Plane (3+ nodes)
│   ├── API Server
│   ├── Scheduler
│   └── Controller Manager
├── Worker Nodes (Auto-scaling)
│   ├── GPU Nodes (Training)
│   ├── CPU Nodes (Inference)
│   └── Storage Nodes
└── Edge Services
    ├── Load Balancer
    ├── Ingress Controller
    └── Monitoring Stack
```

### 2. **Resource Management**
```yaml
# Resource quotas per namespace
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: kubeflow-user-example-com
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    requests.nvidia.com/gpu: "2"
    limits.cpu: "20"
    limits.memory: 40Gi
    limits.nvidia.com/gpu: "4"
    persistentvolumeclaims: "10"
    pods: "20"
```

## 🔐 Security Best Practices

### 1. **Network Security**
```yaml
# Network policies for isolation
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: kubeflow-network-policy
  namespace: kubeflow
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: kubeflow
    - namespaceSelector:
        matchLabels:
          name: istio-system
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: kubeflow
    - namespaceSelector:
        matchLabels:
          name: istio-system
```

### 2. **RBAC Configuration**
```yaml
# Role-based access control
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: kubeflow-user
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps", "secrets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets", "statefulsets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["kubeflow.org"]
  resources: ["notebooks", "pytorchjobs", "tfjobs", "experiments"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

### 3. **Pod Security Standards**
```yaml
# Pod security policy
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: kubeflow-psp
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
  - ALL
  volumes:
  - 'configMap'
  - 'emptyDir'
  - 'projected'
  - 'secret'
  - 'downwardAPI'
  - 'persistentVolumeClaim'
  runAsUser:
    rule: 'MustRunAsNonRoot'
  seLinux:
    rule: 'RunAsAny'
  fsGroup:
    rule: 'RunAsAny'
```

## 📊 Data Management Best Practices

### 1. **Data Versioning Strategy**
```yaml
# Data versioning with DVC integration
apiVersion: v1
kind: ConfigMap
metadata:
  name: data-versioning-config
  namespace: kubeflow
data:
  config.yaml: |
    data_sources:
      raw_data:
        type: "s3"
        bucket: "ml-data-bucket"
        prefix: "raw/"
        versioning: true
      processed_data:
        type: "s3"
        bucket: "ml-data-bucket"
        prefix: "processed/"
        versioning: true
    dvc:
      remote: "s3://ml-data-bucket/dvc-cache"
      cache_dir: "/tmp/dvc-cache"
```

### 2. **Data Quality Checks**
```python
# data_validation.py
import great_expectations as ge
import pandas as pd

def validate_data(df, expectations_suite):
    """Validate data quality using Great Expectations"""
    ge_df = ge.from_pandas(df)
    
    # Load expectations
    suite = ge.ExpectationSuite.load(expectations_suite)
    
    # Validate data
    validation_result = ge_df.validate(expectation_suite=suite)
    
    if not validation_result.success:
        raise ValueError(f"Data validation failed: {validation_result}")
    
    return validation_result
```

### 3. **Data Lineage Tracking**
```yaml
# Pipeline with data lineage
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  name: data-lineage-pipeline
spec:
  entrypoint: data-pipeline
  templates:
  - name: data-pipeline
    dag:
      tasks:
      - name: extract
        template: extract-data
      - name: transform
        template: transform-data
        dependencies: [extract]
      - name: load
        template: load-data
        dependencies: [transform]
```

## 🔄 CI/CD Integration

### 1. **GitOps Workflow**
```yaml
# .github/workflows/kubeflow-ci-cd.yml
name: Kubeflow CI/CD

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
        pip install kfp kubeflow-pipelines-sdk
    
    - name: Test pipeline
      run: |
        python -m pytest tests/
    
    - name: Compile pipeline
      run: |
        python compile_pipeline.py
    
    - name: Deploy to staging
      if: github.ref == 'refs/heads/main'
      run: |
        kubectl apply -f pipeline.yaml -n kubeflow-staging
    
    - name: Deploy to production
      if: github.ref == 'refs/heads/main'
      run: |
        kubectl apply -f pipeline.yaml -n kubeflow-production
```

### 2. **Automated Model Deployment**
```yaml
# model-deployment-pipeline.yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  name: model-deployment
spec:
  entrypoint: deploy-model
  templates:
  - name: deploy-model
    dag:
      tasks:
      - name: validate-model
        template: model-validation
      - name: run-tests
        template: model-tests
        dependencies: [validate-model]
      - name: deploy-staging
        template: deploy-staging
        dependencies: [run-tests]
      - name: integration-tests
        template: integration-tests
        dependencies: [deploy-staging]
      - name: deploy-production
        template: deploy-production
        dependencies: [integration-tests]
```

## 📈 Model Monitoring Best Practices

### 1. **Model Performance Monitoring**
```python
# model_monitoring.py
import mlflow
import pandas as pd
from sklearn.metrics import accuracy_score, precision_score, recall_score
import time

def monitor_model_performance():
    """Monitor model performance in production"""
    
    # Load current model
    model = mlflow.sklearn.load_model("models:/production-model/1")
    
    # Load test data
    test_data = pd.read_csv("data/test.csv")
    
    # Make predictions
    predictions = model.predict(test_data.drop('target', axis=1))
    
    # Calculate metrics
    accuracy = accuracy_score(test_data['target'], predictions)
    precision = precision_score(test_data['target'], predictions, average='weighted')
    recall = recall_score(test_data['target'], predictions, average='weighted')
    
    # Log metrics
    with mlflow.start_run():
        mlflow.log_metric("accuracy", accuracy)
        mlflow.log_metric("precision", precision)
        mlflow.log_metric("recall", recall)
        mlflow.log_metric("timestamp", time.time())
    
    # Alert if performance drops
    if accuracy < 0.8:
        send_alert(f"Model accuracy dropped to {accuracy}")
    
    return {
        "accuracy": accuracy,
        "precision": precision,
        "recall": recall
    }
```

### 2. **Data Drift Detection**
```python
# drift_detection.py
from alibi_detect import TabularDrift
import pandas as pd
import numpy as np

def detect_data_drift(reference_data, new_data, threshold=0.1):
    """Detect data drift between reference and new data"""
    
    # Initialize drift detector
    drift_detector = TabularDrift(
        reference_data.values,
        p_val=0.05
    )
    
    # Detect drift
    drift_score = drift_detector.score(new_data.values)
    
    if drift_score > threshold:
        send_alert(f"Data drift detected! Score: {drift_score}")
        return True
    
    return False
```

### 3. **Model Serving Monitoring**
```yaml
# inference-service with monitoring
apiVersion: serving.kubeflow.org/v1beta1
kind: InferenceService
metadata:
  name: monitored-model
  namespace: kubeflow-user-example-com
spec:
  predictor:
    tensorflow:
      storageUri: gs://my-bucket/models/model-v1
      resources:
        requests:
          cpu: "1"
          memory: "2Gi"
        limits:
          cpu: "2"
          memory: "4Gi"
  transformer:
    custom:
      container:
        image: model-monitor:latest
        env:
        - name: MONITORING_ENABLED
          value: "true"
        - name: ALERT_THRESHOLD
          value: "0.8"
```

## ⚡ Performance Optimization

### 1. **Resource Optimization**
```yaml
# Optimized resource requests
apiVersion: v1
kind: Pod
metadata:
  name: optimized-training-pod
spec:
  containers:
  - name: training
    image: tensorflow/tensorflow:latest
    resources:
      requests:
        cpu: "2"
        memory: "4Gi"
        nvidia.com/gpu: "1"
      limits:
        cpu: "4"
        memory: "8Gi"
        nvidia.com/gpu: "1"
    env:
    - name: TF_FORCE_GPU_ALLOW_GROWTH
      value: "true"
    - name: TF_CPP_MIN_LOG_LEVEL
      value: "2"
```

### 2. **Auto-scaling Configuration**
```yaml
# Horizontal Pod Autoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: model-serving-hpa
  namespace: kubeflow-user-example-com
spec:
  scaleTargetRef:
    apiVersion: serving.kubeflow.org/v1beta1
    kind: InferenceService
    name: my-model
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### 3. **Caching Strategy**
```yaml
# Redis cache for model serving
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-cache
  namespace: kubeflow
spec:
  replicas: 3
  selector:
    matchLabels:
      app: redis-cache
  template:
    metadata:
      labels:
        app: redis-cache
    spec:
      containers:
      - name: redis
        image: redis:alpine
        ports:
        - containerPort: 6379
        resources:
          requests:
            cpu: "0.5"
            memory: "1Gi"
          limits:
            cpu: "1"
            memory: "2Gi"
```

## 🔧 Reproducibility Best Practices

### 1. **Environment Management**
```yaml
# Environment configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: environment-config
  namespace: kubeflow
data:
  config.yaml: |
    environments:
      development:
        cpu: "1"
        memory: "2Gi"
        gpu: "0"
      staging:
        cpu: "2"
        memory: "4Gi"
        gpu: "1"
      production:
        cpu: "4"
        memory: "8Gi"
        gpu: "2"
```

### 2. **Random Seed Management**
```python
# seed_management.py
import random
import numpy as np
import torch
import tensorflow as tf

def set_seeds(seed=42):
    """Set random seeds for reproducibility"""
    random.seed(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)
    if torch.cuda.is_available():
        torch.cuda.manual_seed(seed)
        torch.cuda.manual_seed_all(seed)
    tf.random.set_seed(seed)
    
    # Additional TensorFlow settings
    tf.config.experimental.enable_op_determinism()
```

### 3. **Parameter Management**
```yaml
# Centralized parameter management
apiVersion: v1
kind: ConfigMap
metadata:
  name: model-parameters
  namespace: kubeflow
data:
  params.yaml: |
    model:
      architecture: "resnet50"
      pretrained: true
      num_classes: 10
    
    training:
      batch_size: 32
      learning_rate: 0.001
      epochs: 100
      optimizer: "adam"
      scheduler: "cosine"
    
    data:
      image_size: 224
      augmentation: true
      validation_split: 0.2
```

## 🧪 Testing Best Practices

### 1. **Unit Tests**
```python
# test_pipeline_components.py
import pytest
import pandas as pd
from pipeline_components import prepare_data, train_model

def test_data_preparation():
    """Test data preparation component"""
    # Create test data
    test_data = pd.DataFrame({
        'age': [25, 30, 35],
        'score': [85, 90, 95]
    })
    
    # Test data preparation
    result = prepare_data(test_data)
    
    # Assertions
    assert 'age_normalized' in result.columns
    assert result['age_normalized'].std() == pytest.approx(1.0, rel=1e-2)

def test_model_training():
    """Test model training component"""
    # Create test data
    X = pd.DataFrame({'feature1': [1, 2, 3], 'feature2': [4, 5, 6]})
    y = pd.Series([1, 0, 1])
    
    # Test model training
    model = train_model(X, y)
    
    # Assertions
    assert model is not None
    assert hasattr(model, 'predict')
```

### 2. **Integration Tests**
```python
# test_integration.py
import pytest
import kfp
from kfp import dsl

def test_pipeline_integration():
    """Test complete pipeline integration"""
    
    # Compile pipeline
    pipeline = compile_pipeline()
    
    # Create test run
    client = kfp.Client()
    run = client.create_run_from_pipeline_package(
        pipeline_file="pipeline.yaml",
        arguments={"input_data": "test_data.csv"}
    )
    
    # Wait for completion
    run_result = client.wait_for_run_completion(run.run_id)
    
    # Assertions
    assert run_result.state == "Succeeded"
```

## 📚 Documentation Best Practices

### 1. **Pipeline Documentation**
```python
# pipeline_documentation.py
@dsl.pipeline(
    name="ml-pipeline",
    description="""
    End-to-end ML pipeline for image classification.
    
    This pipeline:
    1. Prepares and validates data
    2. Trains a ResNet50 model
    3. Evaluates model performance
    4. Deploys model for serving
    
    Inputs:
    - input_data: Path to training data
    - model_config: Model configuration parameters
    
    Outputs:
    - trained_model: Trained model artifact
    - metrics: Model performance metrics
    """
)
def ml_pipeline(input_data: str, model_config: dict):
    # Pipeline implementation
    pass
```

### 2. **Component Documentation**
```python
# component_documentation.py
@component(
    packages_to_install=["pandas", "scikit-learn"],
    base_image="python:3.9"
)
def prepare_data(
    input_data: Input[Dataset],
    output_data: Output[Dataset],
    config: dict
) -> None:
    """
    Prepare and preprocess data for training.
    
    Args:
        input_data: Raw input dataset
        output_data: Processed output dataset
        config: Configuration parameters
    
    Returns:
        None
    
    Raises:
        ValueError: If data validation fails
    """
    # Implementation
    pass
```

## 🚨 Common Pitfalls to Avoid

### 1. **Resource Management**
```yaml
# Bad: No resource limits
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: training
    image: tensorflow/tensorflow:latest
    # No resource limits!

# Good: Proper resource management
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: training
    image: tensorflow/tensorflow:latest
    resources:
      requests:
        cpu: "2"
        memory: "4Gi"
      limits:
        cpu: "4"
        memory: "8Gi"
```

### 2. **Security Issues**
```yaml
# Bad: Running as root
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: training
    image: tensorflow/tensorflow:latest
    securityContext:
      runAsUser: 0  # Root user!

# Good: Non-root user
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: training
    image: tensorflow/tensorflow:latest
    securityContext:
      runAsUser: 1000
      runAsNonRoot: true
```

### 3. **Data Management**
```python
# Bad: Hardcoded paths
def load_data():
    df = pd.read_csv("/home/user/data/train.csv")

# Good: Configurable paths
def load_data(data_path: str):
    df = pd.read_csv(data_path)
```

---

*Follow these best practices to build robust, scalable ML systems with Kubeflow! 🎯*