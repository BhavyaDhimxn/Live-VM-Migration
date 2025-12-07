# Kubeflow — Usage

## 🚀 Getting Started with Kubeflow

This guide covers practical usage examples for Kubeflow in real-world ML projects.

## 📊 Example 1: Creating a Jupyter Notebook Server

### Scenario: Data Exploration and Experimentation

Let's create a Jupyter notebook server for data science work.

### Step 1: Access Kubeflow Dashboard
```bash
# Port forward to access dashboard
kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80

# Access at http://localhost:8080
```

### Step 2: Create Notebook Server
```yaml
# notebook-server.yaml
apiVersion: kubeflow.org/v1
kind: Notebook
metadata:
  name: my-notebook
  namespace: kubeflow-user-example-com
spec:
  template:
    spec:
      containers:
      - name: notebook
        image: jupyter/tensorflow-notebook:latest
        resources:
          requests:
            cpu: "1"
            memory: "2Gi"
          limits:
            cpu: "2"
            memory: "4Gi"
        volumeMounts:
        - name: workspace
          mountPath: /home/jovyan/work
      volumes:
      - name: workspace
        persistentVolumeClaim:
          claimName: notebook-pvc
```

### Step 3: Deploy Notebook
```bash
# Apply notebook configuration
kubectl apply -f notebook-server.yaml

# Check notebook status
kubectl get notebooks -n kubeflow-user-example-com

# Access notebook
kubectl port-forward notebook/my-notebook -n kubeflow-user-example-com 8888:8888
```

## 🔧 Example 2: ML Pipeline with Kubeflow Pipelines

### Scenario: End-to-End ML Pipeline

Create a complete ML pipeline with data preparation, training, and serving.

### Step 1: Create Pipeline Components
```python
# pipeline_components.py
import kfp
from kfp import dsl
from kfp.dsl import component, Input, Output, Dataset, Model

@component(
    packages_to_install=["pandas", "scikit-learn"],
    base_image="python:3.9"
)
def prepare_data(
    input_data: Input[Dataset],
    output_data: Output[Dataset]
):
    import pandas as pd
    from sklearn.model_selection import train_test_split
    
    # Load data
    df = pd.read_csv(input_data.path)
    
    # Preprocess data
    df = df.dropna()
    df['age_normalized'] = (df['age'] - df['age'].mean()) / df['age'].std()
    
    # Split data
    train, test = train_test_split(df, test_size=0.2, random_state=42)
    
    # Save processed data
    train.to_csv(f"{output_data.path}/train.csv", index=False)
    test.to_csv(f"{output_data.path}/test.csv", index=False)
    
    print(f"Prepared data: {len(train)} train, {len(test)} test samples")

@component(
    packages_to_install=["pandas", "scikit-learn", "joblib"],
    base_image="python:3.9"
)
def train_model(
    input_data: Input[Dataset],
    model: Output[Model]
):
    import pandas as pd
    import joblib
    from sklearn.linear_model import LinearRegression
    from sklearn.metrics import mean_squared_error, r2_score
    
    # Load data
    train = pd.read_csv(f"{input_data.path}/train.csv")
    test = pd.read_csv(f"{input_data.path}/test.csv")
    
    # Prepare features and target
    X_train = train[['age', 'age_normalized']]
    y_train = train['score']
    X_test = test[['age', 'age_normalized']]
    y_test = test['score']
    
    # Train model
    model_obj = LinearRegression()
    model_obj.fit(X_train, y_train)
    
    # Evaluate model
    y_pred = model_obj.predict(X_test)
    mse = mean_squared_error(y_test, y_pred)
    r2 = r2_score(y_test, y_pred)
    
    # Save model
    joblib.dump(model_obj, f"{model.path}/model.pkl")
    
    print(f"Model trained - MSE: {mse:.2f}, R²: {r2:.2f}")

@component(
    packages_to_install=["pandas", "scikit-learn", "joblib"],
    base_image="python:3.9"
)
def evaluate_model(
    input_data: Input[Dataset],
    model: Input[Model],
    metrics: Output[Dataset]
):
    import pandas as pd
    import joblib
    from sklearn.metrics import mean_squared_error, r2_score
    import json
    
    # Load data and model
    test = pd.read_csv(f"{input_data.path}/test.csv")
    model_obj = joblib.load(f"{model.path}/model.pkl")
    
    # Make predictions
    X_test = test[['age', 'age_normalized']]
    y_test = test['score']
    y_pred = model_obj.predict(X_test)
    
    # Calculate metrics
    mse = mean_squared_error(y_test, y_pred)
    r2 = r2_score(y_test, y_pred)
    
    # Save metrics
    metrics_dict = {
        "mse": float(mse),
        "r2_score": float(r2),
        "test_samples": len(X_test)
    }
    
    with open(f"{metrics.path}/metrics.json", "w") as f:
        json.dump(metrics_dict, f)
    
    print(f"Model evaluation - MSE: {mse:.2f}, R²: {r2:.2f}")
```

### Step 2: Define Pipeline
```python
# pipeline_definition.py
@dsl.pipeline(
    name="ml-pipeline",
    description="End-to-end ML pipeline"
)
def ml_pipeline(
    input_data_path: str = "gs://my-bucket/data/train.csv"
):
    # Data preparation
    prepare_task = prepare_data(
        input_data=dsl.Input(
            type=Dataset,
            uri=input_data_path
        )
    )
    
    # Model training
    train_task = train_model(
        input_data=prepare_task.outputs["output_data"]
    )
    
    # Model evaluation
    evaluate_task = evaluate_model(
        input_data=prepare_task.outputs["output_data"],
        model=train_task.outputs["model"]
    )
    
    # Set resource requirements
    prepare_task.set_cpu_limit("1").set_memory_limit("2Gi")
    train_task.set_cpu_limit("2").set_memory_limit("4Gi")
    evaluate_task.set_cpu_limit("1").set_memory_limit("2Gi")
```

### Step 3: Compile and Run Pipeline
```python
# compile_and_run.py
import kfp

# Compile pipeline
kfp.compiler.Compiler().compile(
    pipeline_func=ml_pipeline,
    package_path="ml_pipeline.yaml"
)

# Create pipeline run
client = kfp.Client()
run = client.create_run_from_pipeline_package(
    pipeline_file="ml_pipeline.yaml",
    arguments={
        "input_data_path": "gs://my-bucket/data/train.csv"
    },
    experiment_name="ml-experiments"
)

print(f"Pipeline run created: {run.run_id}")
```

## 🎯 Example 3: Hyperparameter Tuning with Katib

### Scenario: Optimize Model Hyperparameters

Use Katib to find the best hyperparameters for your model.

### Step 1: Create Katib Experiment
```yaml
# katib-experiment.yaml
apiVersion: kubeflow.org/v1beta1
kind: Experiment
metadata:
  name: hyperparameter-tuning
  namespace: kubeflow-user-example-com
spec:
  algorithm:
    algorithmName: random
  parameters:
  - name: learning-rate
    parameterType: double
    feasibleSpace:
      min: "0.01"
      max: "0.1"
  - name: batch-size
    parameterType: int
    feasibleSpace:
      min: "16"
      max: "128"
  - name: epochs
    parameterType: int
    feasibleSpace:
      min: "50"
      max: "200"
  objective:
    type: minimize
    objectiveMetricName: loss
    additionalMetricNames:
    - accuracy
  maxTrialCount: 20
  parallelTrialCount: 3
  trialTemplate:
    trialSpec:
      apiVersion: batch/v1
      kind: Job
      spec:
        template:
          spec:
            containers:
            - name: training-container
              image: tensorflow/tensorflow:latest
              command:
              - python
              - train.py
              - --learning-rate=${trialParameters.learning-rate}
              - --batch-size=${trialParameters.batch-size}
              - --epochs=${trialParameters.epochs}
              resources:
                requests:
                  cpu: "1"
                  memory: "2Gi"
                  nvidia.com/gpu: "1"
                limits:
                  cpu: "2"
                  memory: "4Gi"
                  nvidia.com/gpu: "1"
            restartPolicy: Never
```

### Step 2: Deploy Experiment
```bash
# Apply Katib experiment
kubectl apply -f katib-experiment.yaml

# Check experiment status
kubectl get experiments -n kubeflow-user-example-com

# Monitor trials
kubectl get trials -n kubeflow-user-example-com
```

### Step 3: View Results
```bash
# Get best trial
kubectl get trials -n kubeflow-user-example-com -o yaml | grep -A 10 "bestTrial"

# Check trial logs
kubectl logs -n kubeflow-user-example-com job/hyperparameter-tuning-trial-1
```

## 🚀 Example 4: Model Serving with KFServing

### Scenario: Deploy Trained Model for Inference

Serve your trained model using KFServing for real-time predictions.

### Step 1: Create Inference Service
```yaml
# inference-service.yaml
apiVersion: serving.kubeflow.org/v1beta1
kind: InferenceService
metadata:
  name: my-model
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
        image: my-model-transformer:latest
        resources:
          requests:
            cpu: "0.5"
            memory: "1Gi"
          limits:
            cpu: "1"
            memory: "2Gi"
```

### Step 2: Deploy Model
```bash
# Apply inference service
kubectl apply -f inference-service.yaml

# Check service status
kubectl get inferenceservice -n kubeflow-user-example-com

# Get service URL
kubectl get inferenceservice my-model -n kubeflow-user-example-com -o jsonpath='{.status.url}'
```

### Step 3: Test Model Inference
```python
# test_inference.py
import requests
import json

# Model endpoint
model_url = "http://my-model.kubeflow-user-example-com.example.com/v1/models/my-model:predict"

# Test data
test_data = {
    "instances": [
        {"age": 25, "age_normalized": 0.5},
        {"age": 30, "age_normalized": 1.0}
    ]
}

# Make prediction
response = requests.post(
    model_url,
    data=json.dumps(test_data),
    headers={"Content-Type": "application/json"}
)

if response.status_code == 200:
    predictions = response.json()
    print(f"Predictions: {predictions}")
else:
    print(f"Error: {response.status_code} - {response.text}")
```

## 🔄 Example 5: Distributed Training with TFJob

### Scenario: Train Large Models with Multiple GPUs

Use TFJob for distributed training across multiple nodes.

### Step 1: Create TFJob
```yaml
# tfjob.yaml
apiVersion: kubeflow.org/v1
kind: TFJob
metadata:
  name: distributed-training
  namespace: kubeflow-user-example-com
spec:
  tfReplicaSpecs:
    Chief:
      replicas: 1
      template:
        spec:
          containers:
          - name: tensorflow
            image: tensorflow/tensorflow:latest
            command:
            - python
            - train.py
            - --job_name=chief
            - --task_index=0
            resources:
              requests:
                cpu: "2"
                memory: "4Gi"
                nvidia.com/gpu: "1"
              limits:
                cpu: "4"
                memory: "8Gi"
                nvidia.com/gpu: "1"
    Worker:
      replicas: 3
      template:
        spec:
          containers:
          - name: tensorflow
            image: tensorflow/tensorflow:latest
            command:
            - python
            - train.py
            - --job_name=worker
            - --task_index=${TASK_INDEX}
            resources:
              requests:
                cpu: "2"
                memory: "4Gi"
                nvidia.com/gpu: "1"
              limits:
                cpu: "4"
                memory: "8Gi"
                nvidia.com/gpu: "1"
```

### Step 2: Deploy Training Job
```bash
# Apply TFJob
kubectl apply -f tfjob.yaml

# Check job status
kubectl get tfjobs -n kubeflow-user-example-com

# Monitor training progress
kubectl logs -n kubeflow-user-example-com -l job-name=distributed-training-chief-0
```

## 📊 Monitoring and Debugging

### Check Pipeline Status
```bash
# List pipeline runs
kubectl get runs -n kubeflow-user-example-com

# Check run details
kubectl describe run ml-pipeline-run-1 -n kubeflow-user-example-com

# View run logs
kubectl logs -n kubeflow-user-example-com -l run-id=ml-pipeline-run-1
```

### Monitor Resource Usage
```bash
# Check pod resources
kubectl top pods -n kubeflow-user-example-com

# Check node resources
kubectl top nodes

# Check GPU usage
kubectl describe nodes | grep -A 5 "nvidia.com/gpu"
```

## 🎯 Common Usage Patterns

### 1. **Data Science Workflow**
```bash
# 1. Create notebook server
kubectl apply -f notebook-server.yaml

# 2. Develop and test code
# (Use Jupyter notebook)

# 3. Create pipeline
kubectl apply -f ml-pipeline.yaml

# 4. Run pipeline
kubectl create -f pipeline-run.yaml
```

### 2. **Model Development**
```bash
# 1. Hyperparameter tuning
kubectl apply -f katib-experiment.yaml

# 2. Train best model
kubectl apply -f tfjob.yaml

# 3. Deploy model
kubectl apply -f inference-service.yaml
```

### 3. **Production Deployment**
```bash
# 1. Create production pipeline
kubectl apply -f production-pipeline.yaml

# 2. Set up monitoring
kubectl apply -f monitoring-config.yaml

# 3. Configure auto-scaling
kubectl apply -f hpa.yaml
```

---

*Kubeflow is now integrated into your ML workflow! 🎉*
