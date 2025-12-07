# MLflow — Usage

## 🚀 Getting Started with MLflow

This guide covers practical usage examples for MLflow in real-world ML projects.

## 📊 Example 1: Basic Experiment Tracking

### Scenario: Track ML Experiment

Let's track a simple machine learning experiment with parameters, metrics, and artifacts.

### Step 1: Setup MLflow
```python
# setup_mlflow.py
import mlflow
import mlflow.sklearn
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, classification_report
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# Set tracking URI
mlflow.set_tracking_uri("http://localhost:5000")

# Create or set experiment
mlflow.set_experiment("my-first-experiment")
```

### Step 2: Create and Track Experiment
```python
# track_experiment.py
with mlflow.start_run() as run:
    # Log parameters
    mlflow.log_param("n_estimators", 100)
    mlflow.log_param("max_depth", 10)
    mlflow.log_param("random_state", 42)
    
    # Create sample data
    X, y = make_classification(
        n_samples=1000, 
        n_features=4, 
        n_redundant=0, 
        n_informative=2, 
        random_state=42
    )
    
    # Split data
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=42
    )
    
    # Train model
    model = RandomForestClassifier(
        n_estimators=100, 
        max_depth=10, 
        random_state=42
    )
    model.fit(X_train, y_train)
    
    # Make predictions
    y_pred = model.predict(X_test)
    
    # Calculate metrics
    accuracy = accuracy_score(y_test, y_pred)
    
    # Log metrics
    mlflow.log_metric("accuracy", accuracy)
    mlflow.log_metric("train_samples", len(X_train))
    mlflow.log_metric("test_samples", len(X_test))
    
    # Log model
    mlflow.sklearn.log_model(
        model, 
        "model",
        registered_model_name="random-forest-model"
    )
    
    # Create and log artifacts
    plt.figure(figsize=(10, 6))
    sns.heatmap(pd.crosstab(y_test, y_pred), annot=True, fmt='d')
    plt.title('Confusion Matrix')
    plt.savefig('confusion_matrix.png')
    mlflow.log_artifact('confusion_matrix.png')
    
    # Log classification report
    report = classification_report(y_test, y_pred, output_dict=True)
    mlflow.log_dict(report, "classification_report.json")
    
    print(f"Run ID: {run.info.run_id}")
    print(f"Accuracy: {accuracy:.4f}")
```

### Step 3: View Results
```python
# view_results.py
# Search runs
runs = mlflow.search_runs(experiment_ids=[1])
print(runs[['run_id', 'metrics.accuracy', 'params.n_estimators']])

# Get best run
best_run = runs.loc[runs['metrics.accuracy'].idxmax()]
print(f"Best accuracy: {best_run['metrics.accuracy']}")
print(f"Best run ID: {best_run['run_id']}")
```

## 🔧 Example 2: MLflow Projects

### Scenario: Reproducible ML Pipeline

Create a reproducible ML pipeline using MLflow Projects.

### Step 1: Create Project Structure
```bash
# Create project directory
mkdir mlflow-project
cd mlflow-project

# Create project files
touch MLproject
mkdir src
touch src/train.py
```

### Step 2: Define MLproject
```yaml
# MLproject
name: mlflow-project
conda_env: conda.yaml
entry_points:
  main:
    parameters:
      n_estimators: {type: int, default: 100}
      max_depth: {type: int, default: 10}
      random_state: {type: int, default: 42}
    command: "python src/train.py --n-estimators {n_estimators} --max-depth {max_depth} --random-state {random_state}"
```

### Step 3: Create Conda Environment
```yaml
# conda.yaml
name: mlflow-project
channels:
  - conda-forge
dependencies:
  - python=3.9
  - scikit-learn=1.0.0
  - pandas=1.3.0
  - matplotlib=3.4.0
  - seaborn=0.11.0
  - pip
  - pip:
    - mlflow
```

### Step 4: Create Training Script
```python
# src/train.py
import argparse
import mlflow
import mlflow.sklearn
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--n-estimators", type=int, default=100)
    parser.add_argument("--max-depth", type=int, default=10)
    parser.add_argument("--random-state", type=int, default=42)
    args = parser.parse_args()
    
    with mlflow.start_run():
        # Log parameters
        mlflow.log_param("n_estimators", args.n_estimators)
        mlflow.log_param("max_depth", args.max_depth)
        mlflow.log_param("random_state", args.random_state)
        
        # Create and split data
        X, y = make_classification(n_samples=1000, n_features=4, random_state=42)
        X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
        
        # Train model
        model = RandomForestClassifier(
            n_estimators=args.n_estimators,
            max_depth=args.max_depth,
            random_state=args.random_state
        )
        model.fit(X_train, y_train)
        
        # Evaluate model
        accuracy = model.score(X_test, y_test)
        mlflow.log_metric("accuracy", accuracy)
        
        # Log model
        mlflow.sklearn.log_model(model, "model")
        
        print(f"Model accuracy: {accuracy:.4f}")

if __name__ == "__main__":
    main()
```

### Step 5: Run Project
```bash
# Run project locally
mlflow run . --experiment-name "mlflow-project-experiment"

# Run with different parameters
mlflow run . -P n_estimators=200 -P max_depth=15

# Run and track in MLflow
mlflow run . --experiment-name "parameter-tuning"
```

## 🚀 Example 3: Model Registry

### Scenario: Model Lifecycle Management

Use MLflow Model Registry to manage model versions and stages.

### Step 1: Register Model
```python
# register_model.py
import mlflow
from mlflow.tracking import MlflowClient

# Set tracking URI
mlflow.set_tracking_uri("http://localhost:5000")

# Create client
client = MlflowClient()

# Register model
model_name = "production-model"
model_version = client.create_registered_model(model_name)
print(f"Registered model: {model_name}")
```

### Step 2: Train and Register Model
```python
# train_and_register.py
import mlflow
import mlflow.sklearn
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score

# Set experiment
mlflow.set_experiment("model-registry-experiment")

with mlflow.start_run() as run:
    # Train model
    X, y = make_classification(n_samples=1000, n_features=4, random_state=42)
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
    
    model = RandomForestClassifier(n_estimators=100, random_state=42)
    model.fit(X_train, y_train)
    
    # Evaluate model
    accuracy = model.score(X_test, y_test)
    mlflow.log_metric("accuracy", accuracy)
    
    # Log and register model
    model_uri = mlflow.sklearn.log_model(
        model, 
        "model",
        registered_model_name="production-model"
    ).model_uri
    
    print(f"Model URI: {model_uri}")
    print(f"Run ID: {run.info.run_id}")
```

### Step 3: Manage Model Versions
```python
# manage_model_versions.py
from mlflow.tracking import MlflowClient

client = MlflowClient()

# List registered models
registered_models = client.search_registered_models()
for model in registered_models:
    print(f"Model: {model.name}")

# Get model versions
model_versions = client.get_latest_versions("production-model")
for version in model_versions:
    print(f"Version: {version.version}, Stage: {version.current_stage}")

# Transition model to staging
client.transition_model_version_stage(
    name="production-model",
    version=1,
    stage="Staging"
)

# Transition model to production
client.transition_model_version_stage(
    name="production-model",
    version=1,
    stage="Production"
)
```

## 🎯 Example 4: Model Serving

### Scenario: Deploy Model for Inference

Serve your trained model using MLflow's built-in serving capabilities.

### Step 1: Serve Model Locally
```bash
# Serve model locally
mlflow models serve -m "models:/production-model/1" -p 5001

# Serve with custom environment
mlflow models serve -m "models:/production-model/1" -p 5001 --env-manager conda

# Serve with custom host
mlflow models serve -m "models:/production-model/1" -p 5001 --host 0.0.0.0
```

### Step 2: Test Model Inference
```python
# test_inference.py
import requests
import json
import numpy as np

# Model endpoint
model_url = "http://localhost:5001/invocations"

# Test data
test_data = {
    "dataframe_split": {
        "columns": ["feature_0", "feature_1", "feature_2", "feature_3"],
        "data": [[0.1, 0.2, 0.3, 0.4], [0.5, 0.6, 0.7, 0.8]]
    }
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

### Step 3: Deploy to Cloud
```python
# deploy_to_cloud.py
import mlflow.sagemaker

# Deploy to AWS SageMaker
mlflow.sagemaker.deploy(
    app_name="my-model-endpoint",
    model_uri="models:/production-model/1",
    execution_role_arn="arn:aws:iam::123456789012:role/MLflowSageMakerRole",
    region_name="us-west-2"
)
```

## 🔄 Example 5: Hyperparameter Tuning

### Scenario: Optimize Model Hyperparameters

Use MLflow to track hyperparameter tuning experiments.

### Step 1: Create Hyperparameter Tuning Script
```python
# hyperparameter_tuning.py
import mlflow
import mlflow.sklearn
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split, GridSearchCV
from sklearn.metrics import accuracy_score
import itertools

# Set experiment
mlflow.set_experiment("hyperparameter-tuning")

# Define parameter grid
param_grid = {
    'n_estimators': [50, 100, 200],
    'max_depth': [5, 10, 15],
    'min_samples_split': [2, 5, 10]
}

# Create data
X, y = make_classification(n_samples=1000, n_features=4, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Generate all parameter combinations
param_combinations = list(itertools.product(*param_grid.values()))

for i, params in enumerate(param_combinations):
    with mlflow.start_run() as run:
        # Log parameters
        for param_name, param_value in zip(param_grid.keys(), params):
            mlflow.log_param(param_name, param_value)
        
        # Train model
        model = RandomForestClassifier(
            n_estimators=params[0],
            max_depth=params[1],
            min_samples_split=params[2],
            random_state=42
        )
        model.fit(X_train, y_train)
        
        # Evaluate model
        accuracy = model.score(X_test, y_test)
        mlflow.log_metric("accuracy", accuracy)
        
        # Log model
        mlflow.sklearn.log_model(model, "model")
        
        print(f"Run {i+1}/{len(param_combinations)}: Accuracy = {accuracy:.4f}")
```

### Step 2: Analyze Results
```python
# analyze_results.py
import mlflow
import pandas as pd

# Search runs
runs = mlflow.search_runs(experiment_ids=[2])

# Get best run
best_run = runs.loc[runs['metrics.accuracy'].idxmax()]
print(f"Best accuracy: {best_run['metrics.accuracy']}")
print(f"Best parameters:")
print(f"  n_estimators: {best_run['params.n_estimators']}")
print(f"  max_depth: {best_run['params.max_depth']}")
print(f"  min_samples_split: {best_run['params.min_samples_split']}")

# Create parameter importance plot
import matplotlib.pyplot as plt

fig, axes = plt.subplots(1, 3, figsize=(15, 5))

for i, param in enumerate(['n_estimators', 'max_depth', 'min_samples_split']):
    param_values = runs[f'params.{param}'].astype(float)
    accuracies = runs['metrics.accuracy']
    
    axes[i].scatter(param_values, accuracies, alpha=0.6)
    axes[i].set_xlabel(param)
    axes[i].set_ylabel('Accuracy')
    axes[i].set_title(f'{param} vs Accuracy')

plt.tight_layout()
plt.savefig('parameter_importance.png')
plt.show()
```

## 📊 Monitoring and Debugging

### Check Experiment Status
```python
# Check experiment status
import mlflow

# List experiments
experiments = mlflow.search_experiments()
for exp in experiments:
    print(f"Experiment: {exp.name}, ID: {exp.experiment_id}")

# Search runs
runs = mlflow.search_runs(experiment_ids=[1])
print(f"Total runs: {len(runs)}")
print(f"Best accuracy: {runs['metrics.accuracy'].max()}")
```

### Monitor Model Performance
```python
# Monitor model performance
from mlflow.tracking import MlflowClient

client = MlflowClient()

# Get model versions
model_versions = client.get_latest_versions("production-model")
for version in model_versions:
    print(f"Version: {version.version}, Stage: {version.current_stage}")
    
    # Get run details
    run = client.get_run(version.run_id)
    print(f"  Accuracy: {run.data.metrics['accuracy']}")
    print(f"  Parameters: {run.data.params}")
```

## 🎯 Common Usage Patterns

### 1. **Data Science Workflow**
```python
# 1. Set experiment
mlflow.set_experiment("data-science-workflow")

# 2. Track experiments
with mlflow.start_run():
    # Log parameters, metrics, and artifacts
    pass

# 3. Compare results
runs = mlflow.search_runs()
best_run = runs.loc[runs['metrics.accuracy'].idxmax()]

# 4. Register best model
mlflow.sklearn.log_model(model, "model", registered_model_name="best-model")
```

### 2. **Model Development**
```python
# 1. Hyperparameter tuning
# (Use hyperparameter_tuning.py)

# 2. Register best model
# (Use register_model.py)

# 3. Deploy model
# (Use deploy_to_cloud.py)
```

### 3. **Production Deployment**
```python
# 1. Load production model
model = mlflow.pyfunc.load_model("models:/production-model/1")

# 2. Make predictions
predictions = model.predict(new_data)

# 3. Monitor performance
# (Use monitoring scripts)
```

---

*MLflow is now integrated into your ML workflow! 🎉*
