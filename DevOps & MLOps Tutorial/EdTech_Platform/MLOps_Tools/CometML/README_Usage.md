# CometML — Usage

## 🚀 Getting Started with CometML

This guide covers practical usage examples for CometML in real-world ML projects.

## 📊 Example 1: Basic Experiment Tracking

### Scenario: Track ML Experiment

Let's track a simple machine learning experiment with parameters, metrics, and artifacts.

### Step 1: Setup CometML
```python
# setup_comet.py
import comet_ml
import numpy as np
import pandas as pd
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, classification_report
import matplotlib.pyplot as plt
import seaborn as sns

# Initialize experiment
experiment = comet_ml.Experiment(
    api_key="your-api-key",
    project_name="my-first-project",
    workspace="my-workspace"
)
```

### Step 2: Create and Track Experiment
```python
# track_experiment.py
# Log experiment metadata
experiment.set_name("Random Forest Classification")
experiment.add_tags(["classification", "random-forest", "tutorial"])

# Log parameters
experiment.log_parameter("n_estimators", 100)
experiment.log_parameter("max_depth", 10)
experiment.log_parameter("random_state", 42)
experiment.log_parameter("test_size", 0.2)

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

# Log data information
experiment.log_parameter("train_samples", len(X_train))
experiment.log_parameter("test_samples", len(X_test))
experiment.log_parameter("features", X.shape[1])

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
experiment.log_metric("accuracy", accuracy)
experiment.log_metric("train_samples", len(X_train))
experiment.log_metric("test_samples", len(X_test))

# Log model
experiment.log_model(model, "random_forest_model")

# Create and log confusion matrix
plt.figure(figsize=(10, 6))
sns.heatmap(pd.crosstab(y_test, y_pred), annot=True, fmt='d')
plt.title('Confusion Matrix')
experiment.log_figure(plt.gcf(), "confusion_matrix")

# Log classification report
report = classification_report(y_test, y_pred, output_dict=True)
experiment.log_metrics(report)

# Log feature importance
feature_importance = model.feature_importances_
experiment.log_other("feature_importance", feature_importance.tolist())

print(f"Experiment ID: {experiment.get_key()}")
print(f"Accuracy: {accuracy:.4f}")

# End experiment
experiment.end()
```

### Step 3: View Results
```python
# view_results.py
import comet_ml

# Get experiment details
api = comet_ml.API()
experiment = api.get_experiment("experiment-id")

# Get experiment metrics
metrics = experiment.get_metrics()
print("Metrics:", metrics)

# Get experiment parameters
parameters = experiment.get_parameters()
print("Parameters:", parameters)

# Get experiment assets
assets = experiment.get_assets()
print("Assets:", assets)
```

## 🔧 Example 2: Hyperparameter Optimization

### Scenario: Optimize Model Hyperparameters

Use CometML's Optuna integration for automated hyperparameter tuning.

### Step 1: Setup Hyperparameter Optimization
```python
# hyperparameter_optimization.py
import comet_ml
from comet_ml import Optimizer
import optuna
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.metrics import accuracy_score
import numpy as np

# Initialize optimizer
opt = Optimizer(
    api_key="your-api-key",
    project_name="hyperparameter-optimization"
)

# Define search space
config = {
    "n_estimators": [50, 100, 200, 300],
    "max_depth": [5, 10, 15, 20, None],
    "min_samples_split": [2, 5, 10],
    "min_samples_leaf": [1, 2, 4],
    "max_features": ["sqrt", "log2", None]
}

# Create data
X, y = make_classification(
    n_samples=1000, 
    n_features=4, 
    random_state=42
)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
```

### Step 2: Run Optimization
```python
# Run hyperparameter optimization
for experiment in opt.get_experiments(config):
    # Get parameters
    n_estimators = experiment.get_parameter("n_estimators")
    max_depth = experiment.get_parameter("max_depth")
    min_samples_split = experiment.get_parameter("min_samples_split")
    min_samples_leaf = experiment.get_parameter("min_samples_leaf")
    max_features = experiment.get_parameter("max_features")
    
    # Train model
    model = RandomForestClassifier(
        n_estimators=n_estimators,
        max_depth=max_depth,
        min_samples_split=min_samples_split,
        min_samples_leaf=min_samples_leaf,
        max_features=max_features,
        random_state=42
    )
    
    # Cross-validation
    cv_scores = cross_val_score(model, X_train, y_train, cv=5)
    mean_cv_score = cv_scores.mean()
    std_cv_score = cv_scores.std()
    
    # Test evaluation
    model.fit(X_train, y_train)
    test_accuracy = model.score(X_test, y_test)
    
    # Log results
    experiment.log_metric("cv_score_mean", mean_cv_score)
    experiment.log_metric("cv_score_std", std_cv_score)
    experiment.log_metric("test_accuracy", test_accuracy)
    
    print(f"CV Score: {mean_cv_score:.4f} (+/- {std_cv_score:.4f})")
    print(f"Test Accuracy: {test_accuracy:.4f}")
```

### Step 3: Analyze Results
```python
# analyze_optimization_results.py
import comet_ml
import pandas as pd
import matplotlib.pyplot as plt

# Get optimization results
api = comet_ml.API()
experiments = api.get_experiments(
    workspace="my-workspace",
    project_name="hyperparameter-optimization"
)

# Create results DataFrame
results = []
for exp in experiments:
    result = {
        "experiment_id": exp.id,
        "n_estimators": exp.get_parameter("n_estimators"),
        "max_depth": exp.get_parameter("max_depth"),
        "min_samples_split": exp.get_parameter("min_samples_split"),
        "min_samples_leaf": exp.get_parameter("min_samples_leaf"),
        "max_features": exp.get_parameter("max_features"),
        "cv_score": exp.get_metric("cv_score_mean"),
        "test_accuracy": exp.get_metric("test_accuracy")
    }
    results.append(result)

df = pd.DataFrame(results)

# Find best experiment
best_experiment = df.loc[df["cv_score"].idxmax()]
print("Best Parameters:")
print(best_experiment)

# Create parameter importance plot
fig, axes = plt.subplots(2, 3, figsize=(15, 10))
axes = axes.ravel()

for i, param in enumerate(["n_estimators", "max_depth", "min_samples_split", 
                          "min_samples_leaf", "max_features"]):
    if i < len(axes):
        axes[i].scatter(df[param], df["cv_score"], alpha=0.6)
        axes[i].set_xlabel(param)
        axes[i].set_ylabel("CV Score")
        axes[i].set_title(f"{param} vs CV Score")

plt.tight_layout()
plt.savefig("parameter_importance.png")
plt.show()
```

## 🚀 Example 3: Model Registry and Deployment

### Scenario: Register and Deploy Model

Use CometML's model registry to manage model versions and deploy to production.

### Step 1: Register Model
```python
# register_model.py
import comet_ml
import joblib
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score

# Train model
X, y = make_classification(n_samples=1000, n_features=4, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)

# Evaluate model
accuracy = model.score(X_test, y_test)

# Initialize experiment
experiment = comet_ml.Experiment(
    api_key="your-api-key",
    project_name="model-registry"
)

# Log model and metrics
experiment.log_model(model, "production-model")
experiment.log_metric("accuracy", accuracy)
experiment.log_parameter("n_estimators", 100)
experiment.log_parameter("random_state", 42)

# End experiment
experiment.end()

# Register model in registry
api = comet_ml.API()
model = api.get_model(
    workspace="my-workspace",
    model_name="production-model"
)

# Add model version
model.add_version(
    version="1.0.0",
    path="path/to/model.pkl",
    metadata={
        "accuracy": accuracy,
        "n_estimators": 100,
        "training_date": "2023-01-01"
    }
)

print(f"Model registered: {model.name}")
print(f"Version: 1.0.0")
print(f"Accuracy: {accuracy:.4f}")
```

### Step 2: Deploy Model
```python
# deploy_model.py
import comet_ml
import requests
import json

# Get model from registry
api = comet_ml.API()
model = api.get_model(
    workspace="my-workspace",
    model_name="production-model"
)

# Deploy to AWS SageMaker
deployment = model.deploy(
    platform="sagemaker",
    endpoint_name="my-model-endpoint",
    instance_type="ml.m5.large",
    region="us-west-2"
)

print(f"Model deployed to: {deployment.endpoint_url}")

# Test deployment
test_data = {
    "data": [[0.1, 0.2, 0.3, 0.4], [0.5, 0.6, 0.7, 0.8]]
}

response = requests.post(
    deployment.endpoint_url,
    json=test_data,
    headers={"Content-Type": "application/json"}
)

if response.status_code == 200:
    predictions = response.json()
    print(f"Predictions: {predictions}")
else:
    print(f"Error: {response.status_code} - {response.text}")
```

## 🎯 Example 4: Model Monitoring

### Scenario: Monitor Production Model

Use CometML's model monitoring to track production model performance.

### Step 1: Setup Model Monitoring
```python
# model_monitoring.py
import comet_ml
import numpy as np
import pandas as pd
from sklearn.metrics import accuracy_score, precision_score, recall_score
import time
from datetime import datetime

# Initialize model monitor
monitor = comet_ml.ModelMonitor(
    model_id="model-id",
    api_key="your-api-key"
)

# Simulate production data
def simulate_production_data():
    """Simulate incoming production data"""
    np.random.seed(int(time.time()))
    X = np.random.rand(100, 4)
    y = np.random.randint(0, 2, 100)
    return X, y

# Monitor model performance
def monitor_model_performance():
    """Monitor model performance in production"""
    
    # Get production data
    X_prod, y_true = simulate_production_data()
    
    # Load model (in real scenario, this would be from your serving endpoint)
    # For demo, we'll use a simple model
    from sklearn.ensemble import RandomForestClassifier
    model = RandomForestClassifier(n_estimators=100, random_state=42)
    
    # Make predictions
    y_pred = model.predict(X_prod)
    
    # Calculate metrics
    accuracy = accuracy_score(y_true, y_pred)
    precision = precision_score(y_true, y_pred, average='weighted')
    recall = recall_score(y_true, y_pred, average='weighted')
    
    # Log predictions
    monitor.log_predictions(
        inputs=X_prod.tolist(),
        outputs=y_pred.tolist(),
        metadata={
            "timestamp": datetime.now().isoformat(),
            "batch_size": len(X_prod)
        }
    )
    
    # Log metrics
    monitor.log_metrics({
        "accuracy": accuracy,
        "precision": precision,
        "recall": recall,
        "timestamp": time.time()
    })
    
    # Log data statistics
    monitor.log_metrics({
        "input_mean": X_prod.mean(),
        "input_std": X_prod.std(),
        "output_distribution": np.bincount(y_pred).tolist()
    })
    
    print(f"Accuracy: {accuracy:.4f}")
    print(f"Precision: {precision:.4f}")
    print(f"Recall: {recall:.4f}")
    
    return {
        "accuracy": accuracy,
        "precision": precision,
        "recall": recall
    }

# Run monitoring
for i in range(10):
    print(f"Monitoring batch {i+1}")
    monitor_model_performance()
    time.sleep(5)  # Wait 5 seconds between batches
```

### Step 2: Analyze Monitoring Data
```python
# analyze_monitoring_data.py
import comet_ml
import pandas as pd
import matplotlib.pyplot as plt

# Get monitoring data
api = comet_ml.API()
monitor_data = api.get_model_monitor_data("model-id")

# Create monitoring DataFrame
monitoring_df = pd.DataFrame(monitor_data)

# Plot performance over time
plt.figure(figsize=(12, 8))

plt.subplot(2, 2, 1)
plt.plot(monitoring_df["timestamp"], monitoring_df["accuracy"])
plt.title("Accuracy Over Time")
plt.xlabel("Time")
plt.ylabel("Accuracy")

plt.subplot(2, 2, 2)
plt.plot(monitoring_df["timestamp"], monitoring_df["precision"])
plt.title("Precision Over Time")
plt.xlabel("Time")
plt.ylabel("Precision")

plt.subplot(2, 2, 3)
plt.plot(monitoring_df["timestamp"], monitoring_df["recall"])
plt.title("Recall Over Time")
plt.xlabel("Time")
plt.ylabel("Recall")

plt.subplot(2, 2, 4)
plt.plot(monitoring_df["timestamp"], monitoring_df["input_mean"])
plt.title("Input Mean Over Time")
plt.xlabel("Time")
plt.ylabel("Input Mean")

plt.tight_layout()
plt.savefig("monitoring_plots.png")
plt.show()

# Detect performance degradation
accuracy_threshold = 0.8
degraded_batches = monitoring_df[monitoring_df["accuracy"] < accuracy_threshold]

if len(degraded_batches) > 0:
    print(f"Performance degradation detected in {len(degraded_batches)} batches")
    print("Degraded batches:")
    print(degraded_batches[["timestamp", "accuracy"]])
else:
    print("No performance degradation detected")
```

## 🔄 Example 5: Team Collaboration

### Scenario: Collaborative ML Project

Use CometML for team collaboration on ML projects.

### Step 1: Setup Team Workspace
```python
# team_collaboration.py
import comet_ml
import pandas as pd
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split

# Initialize experiment with team metadata
experiment = comet_ml.Experiment(
    api_key="your-api-key",
    project_name="team-ml-project",
    workspace="team-workspace"
)

# Add team information
experiment.log_other("team_members", ["alice", "bob", "charlie"])
experiment.log_other("project_lead", "alice")
experiment.log_other("sprint", "sprint-1")

# Add experiment description
experiment.log_other("description", "Collaborative ML project for customer churn prediction")

# Train model
X, y = make_classification(n_samples=1000, n_features=4, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)

# Log results
accuracy = model.score(X_test, y_test)
experiment.log_metric("accuracy", accuracy)
experiment.log_model(model, "team-model")

# Add comments
experiment.log_other("comments", [
    "Initial model with basic features",
    "Need to add more feature engineering",
    "Consider ensemble methods"
])

experiment.end()
```

### Step 2: Review Team Experiments
```python
# review_team_experiments.py
import comet_ml
import pandas as pd

# Get team experiments
api = comet_ml.API()
experiments = api.get_experiments(
    workspace="team-workspace",
    project_name="team-ml-project"
)

# Create team results DataFrame
team_results = []
for exp in experiments:
    result = {
        "experiment_id": exp.id,
        "experiment_name": exp.name,
        "accuracy": exp.get_metric("accuracy"),
        "team_members": exp.get_other("team_members"),
        "sprint": exp.get_other("sprint"),
        "comments": exp.get_other("comments")
    }
    team_results.append(result)

df = pd.DataFrame(team_results)

# Display team results
print("Team Experiment Results:")
print(df[["experiment_name", "accuracy", "sprint"]])

# Find best experiment
best_experiment = df.loc[df["accuracy"].idxmax()]
print(f"\nBest Experiment: {best_experiment['experiment_name']}")
print(f"Accuracy: {best_experiment['accuracy']:.4f}")

# Group by sprint
sprint_results = df.groupby("sprint")["accuracy"].agg(["mean", "max", "count"])
print("\nSprint Results:")
print(sprint_results)
```

## 📊 Monitoring and Debugging

### Check Experiment Status
```python
# Check experiment status
import comet_ml

api = comet_ml.API()
experiment = api.get_experiment("experiment-id")

print(f"Experiment: {experiment.name}")
print(f"Status: {experiment.status}")
print(f"Metrics: {experiment.get_metrics()}")
print(f"Parameters: {experiment.get_parameters()}")
```

### Monitor Model Performance
```python
# Monitor model performance
monitor = comet_ml.ModelMonitor("model-id")

# Get monitoring data
monitoring_data = monitor.get_data()

# Analyze performance trends
import pandas as pd
df = pd.DataFrame(monitoring_data)
print(f"Average accuracy: {df['accuracy'].mean():.4f}")
print(f"Accuracy std: {df['accuracy'].std():.4f}")
```

## 🎯 Common Usage Patterns

### 1. **Data Science Workflow**
```python
# 1. Initialize experiment
experiment = comet_ml.Experiment(project_name="data-science-workflow")

# 2. Log data and parameters
experiment.log_parameter("dataset", "customer-data")
experiment.log_parameter("model_type", "random-forest")

# 3. Train and evaluate model
# (Training code here)

# 4. Log results
experiment.log_metric("accuracy", accuracy)
experiment.log_model(model, "final-model")

# 5. End experiment
experiment.end()
```

### 2. **Model Development**
```python
# 1. Hyperparameter optimization
# (Use Optimizer class)

# 2. Register best model
# (Use model registry)

# 3. Deploy model
# (Use deployment features)

# 4. Monitor model
# (Use model monitoring)
```

### 3. **Production Deployment**
```python
# 1. Load production model
model = api.get_model("production-model")

# 2. Deploy to cloud
deployment = model.deploy("sagemaker", "endpoint-name")

# 3. Monitor performance
monitor = comet_ml.ModelMonitor("model-id")
monitor.log_predictions(inputs, outputs)
```

---

*CometML is now integrated into your ML workflow! 🎉*