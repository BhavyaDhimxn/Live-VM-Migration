# WandB — Usage

## 🚀 Getting Started with WandB

This guide covers practical usage examples for WandB in real-world ML projects.

## 📊 Example 1: Basic Experiment Tracking

### Scenario: Track ML Experiment

Let's track a simple machine learning experiment with parameters, metrics, and artifacts.

### Step 1: Setup WandB
```python
# setup_wandb.py
import wandb
import numpy as np
import pandas as pd
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, classification_report
import matplotlib.pyplot as plt
import seaborn as sns

# Initialize wandb
wandb.init(
    project="my-first-project",
    name="random-forest-experiment",
    config={
        "n_estimators": 100,
        "max_depth": 10,
        "random_state": 42,
        "test_size": 0.2
    }
)
```

### Step 2: Create and Track Experiment
```python
# track_experiment.py
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
wandb.config.update({
    "train_samples": len(X_train),
    "test_samples": len(X_test),
    "features": X.shape[1]
})

# Train model
model = RandomForestClassifier(
    n_estimators=wandb.config.n_estimators,
    max_depth=wandb.config.max_depth,
    random_state=wandb.config.random_state
)
model.fit(X_train, y_train)

# Make predictions
y_pred = model.predict(X_test)

# Calculate metrics
accuracy = accuracy_score(y_test, y_pred)
report = classification_report(y_test, y_pred, output_dict=True)

# Log metrics
wandb.log({
    "accuracy": accuracy,
    "train_samples": len(X_train),
    "test_samples": len(X_test)
})

# Log classification report
wandb.log({
    "precision": report["weighted avg"]["precision"],
    "recall": report["weighted avg"]["recall"],
    "f1_score": report["weighted avg"]["f1-score"]
})

# Log model
wandb.log_model(model, "random_forest_model")

# Create and log confusion matrix
plt.figure(figsize=(10, 6))
sns.heatmap(pd.crosstab(y_test, y_pred), annot=True, fmt='d')
plt.title('Confusion Matrix')
wandb.log({"confusion_matrix": wandb.Image(plt)})

# Log feature importance
feature_importance = model.feature_importances_
wandb.log({
    "feature_importance": wandb.plot.bar(
        wandb.Table(
            data=[[f"feature_{i}", imp] for i, imp in enumerate(feature_importance)],
            columns=["feature", "importance"]
        ),
        "feature", "importance"
    )
})

print(f"Run ID: {wandb.run.id}")
print(f"Accuracy: {accuracy:.4f}")

# Finish run
wandb.finish()
```

### Step 3: View Results
```python
# view_results.py
import wandb

# Get API instance
api = wandb.Api()

# Get runs from project
runs = api.runs("my-first-project")

# Display runs
for run in runs:
    print(f"Run: {run.name}")
    print(f"Accuracy: {run.summary.get('accuracy', 'N/A')}")
    print(f"Config: {run.config}")
    print("---")

# Get best run
best_run = max(runs, key=lambda x: x.summary.get('accuracy', 0))
print(f"\nBest Run: {best_run.name}")
print(f"Best Accuracy: {best_run.summary['accuracy']}")
```

## 🔧 Example 2: Hyperparameter Sweeps

### Scenario: Optimize Model Hyperparameters

Use WandB Sweeps for automated hyperparameter tuning.

### Step 1: Define Sweep Configuration
```python
# hyperparameter_sweep.py
import wandb
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.metrics import accuracy_score
import numpy as np

# Define sweep configuration
sweep_config = {
    "method": "bayes",  # or "random", "grid"
    "metric": {
        "name": "cv_score",
        "goal": "maximize"
    },
    "parameters": {
        "n_estimators": {
            "distribution": "int_uniform",
            "min": 50,
            "max": 300
        },
        "max_depth": {
            "distribution": "int_uniform",
            "min": 5,
            "max": 20
        },
        "min_samples_split": {
            "values": [2, 5, 10]
        },
        "min_samples_leaf": {
            "values": [1, 2, 4]
        },
        "max_features": {
            "values": ["sqrt", "log2", None]
        }
    }
}

# Create sweep
sweep_id = wandb.sweep(sweep_config, project="hyperparameter-optimization")
```

### Step 2: Define Training Function
```python
# Training function for sweep
def train():
    # Initialize wandb run
    wandb.init()
    
    # Get hyperparameters from sweep
    config = wandb.config
    
    # Create data
    X, y = make_classification(
        n_samples=1000, 
        n_features=4, 
        random_state=42
    )
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=42
    )
    
    # Train model with sweep parameters
    model = RandomForestClassifier(
        n_estimators=config.n_estimators,
        max_depth=config.max_depth,
        min_samples_split=config.min_samples_split,
        min_samples_leaf=config.min_samples_leaf,
        max_features=config.max_features,
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
    wandb.log({
        "cv_score": mean_cv_score,
        "cv_score_std": std_cv_score,
        "test_accuracy": test_accuracy
    })
    
    print(f"CV Score: {mean_cv_score:.4f} (+/- {std_cv_score:.4f})")
    print(f"Test Accuracy: {test_accuracy:.4f}")
```

### Step 3: Run Sweep
```python
# Run sweep
wandb.agent(sweep_id, train, count=50)  # Run 50 iterations
```

### Step 4: Analyze Sweep Results
```python
# analyze_sweep_results.py
import wandb
import pandas as pd
import matplotlib.pyplot as plt

# Get sweep results
api = wandb.Api()
sweep = api.sweep("hyperparameter-optimization/sweep-id")

# Create results DataFrame
runs = sweep.runs
results = []
for run in runs:
    result = {
        "run_id": run.id,
        "n_estimators": run.config.get("n_estimators"),
        "max_depth": run.config.get("max_depth"),
        "min_samples_split": run.config.get("min_samples_split"),
        "min_samples_leaf": run.config.get("min_samples_leaf"),
        "max_features": run.config.get("max_features"),
        "cv_score": run.summary.get("cv_score"),
        "test_accuracy": run.summary.get("test_accuracy")
    }
    results.append(result)

df = pd.DataFrame(results)

# Find best run
best_run = df.loc[df["cv_score"].idxmax()]
print("Best Parameters:")
print(best_run)

# Create parameter importance plots
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

Use WandB's model registry to manage model versions and deploy to production.

### Step 1: Register Model
```python
# register_model.py
import wandb
import joblib
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split

# Train model
X, y = make_classification(n_samples=1000, n_features=4, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)

# Evaluate model
accuracy = model.score(X_test, y_test)

# Initialize wandb
wandb.init(
    project="model-registry",
    name="production-model-v1"
)

# Log model and metrics
wandb.log({"accuracy": accuracy})
wandb.config.update({
    "n_estimators": 100,
    "random_state": 42
})

# Register model in registry
wandb.log_model(
    model,
    "production-model",
    aliases=["latest", "production", "v1.0.0"]
)

# Add metadata
wandb.log({
    "model_metadata": {
        "accuracy": accuracy,
        "n_estimators": 100,
        "training_date": "2023-01-01"
    }
})

wandb.finish()

print(f"Model registered: production-model")
print(f"Version: v1.0.0")
print(f"Accuracy: {accuracy:.4f}")
```

### Step 2: Deploy Model
```python
# deploy_model.py
import wandb
import requests
import json

# Get model from registry
api = wandb.Api()
model = api.artifact("model-registry/production-model:latest")

# Download model
model_dir = model.download()
model_path = f"{model_dir}/model.pkl"

# Load model
import joblib
loaded_model = joblib.load(model_path)

# Deploy to AWS SageMaker (example)
# This would typically use boto3 or SageMaker SDK
# For demonstration, we'll show the structure

def deploy_to_sagemaker(model_path, endpoint_name):
    """Deploy model to AWS SageMaker"""
    # Implementation would use boto3
    # sagemaker_client.create_endpoint(...)
    pass

# Test model locally first
test_data = [[0.1, 0.2, 0.3, 0.4], [0.5, 0.6, 0.7, 0.8]]
predictions = loaded_model.predict(test_data)
print(f"Predictions: {predictions}")
```

## 🎯 Example 4: Model Monitoring

### Scenario: Monitor Production Model

Use WandB to track production model performance.

### Step 1: Setup Model Monitoring
```python
# model_monitoring.py
import wandb
import numpy as np
import pandas as pd
from sklearn.metrics import accuracy_score, precision_score, recall_score
import time
from datetime import datetime

# Initialize monitoring run
wandb.init(
    project="model-monitoring",
    name="production-monitoring"
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
    from sklearn.ensemble import RandomForestClassifier
    model = RandomForestClassifier(n_estimators=100, random_state=42)
    model.fit(X_prod, y_true)  # In production, this would be pre-trained
    
    # Make predictions
    y_pred = model.predict(X_prod)
    
    # Calculate metrics
    accuracy = accuracy_score(y_true, y_pred)
    precision = precision_score(y_true, y_pred, average='weighted')
    recall = recall_score(y_true, y_pred, average='weighted')
    
    # Log predictions
    wandb.log({
        "predictions": y_pred.tolist(),
        "ground_truth": y_true.tolist(),
        "timestamp": time.time()
    })
    
    # Log metrics
    wandb.log({
        "accuracy": accuracy,
        "precision": precision,
        "recall": recall,
        "batch_size": len(X_prod)
    })
    
    # Log data statistics
    wandb.log({
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

wandb.finish()
```

### Step 2: Analyze Monitoring Data
```python
# analyze_monitoring_data.py
import wandb
import pandas as pd
import matplotlib.pyplot as plt

# Get monitoring data
api = wandb.Api()
runs = api.runs("model-monitoring")

# Create monitoring DataFrame
monitoring_data = []
for run in runs:
    for log in run.history():
        monitoring_data.append({
            "timestamp": log.get("_timestamp"),
            "accuracy": log.get("accuracy"),
            "precision": log.get("precision"),
            "recall": log.get("recall"),
            "input_mean": log.get("input_mean")
        })

df = pd.DataFrame(monitoring_data)

# Plot performance over time
fig, axes = plt.subplots(2, 2, figsize=(12, 8))

axes[0, 0].plot(df["timestamp"], df["accuracy"])
axes[0, 0].set_title("Accuracy Over Time")
axes[0, 0].set_xlabel("Time")
axes[0, 0].set_ylabel("Accuracy")

axes[0, 1].plot(df["timestamp"], df["precision"])
axes[0, 1].set_title("Precision Over Time")
axes[0, 1].set_xlabel("Time")
axes[0, 1].set_ylabel("Precision")

axes[1, 0].plot(df["timestamp"], df["recall"])
axes[1, 0].set_title("Recall Over Time")
axes[1, 0].set_xlabel("Time")
axes[1, 0].set_ylabel("Recall")

axes[1, 1].plot(df["timestamp"], df["input_mean"])
axes[1, 1].set_title("Input Mean Over Time")
axes[1, 1].set_xlabel("Time")
axes[1, 1].set_ylabel("Input Mean")

plt.tight_layout()
plt.savefig("monitoring_plots.png")
plt.show()

# Detect performance degradation
accuracy_threshold = 0.8
degraded_batches = df[df["accuracy"] < accuracy_threshold]

if len(degraded_batches) > 0:
    print(f"Performance degradation detected in {len(degraded_batches)} batches")
    print("Degraded batches:")
    print(degraded_batches[["timestamp", "accuracy"]])
else:
    print("No performance degradation detected")
```

## 🔄 Example 5: Team Collaboration

### Scenario: Collaborative ML Project

Use WandB for team collaboration on ML projects.

### Step 1: Setup Team Workspace
```python
# team_collaboration.py
import wandb
import pandas as pd
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split

# Initialize experiment with team metadata
wandb.init(
    project="team-ml-project",
    name="alice-experiment-1",
    tags=["team-project", "sprint-1", "alice"],
    notes="Collaborative ML project for customer churn prediction"
)

# Add team information
wandb.config.update({
    "team_members": ["alice", "bob", "charlie"],
    "project_lead": "alice",
    "sprint": "sprint-1"
})

# Train model
X, y = make_classification(n_samples=1000, n_features=4, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)

# Log results
accuracy = model.score(X_test, y_test)
wandb.log({"accuracy": accuracy})
wandb.log_model(model, "team-model")

# Add comments
wandb.log({
    "comments": [
        "Initial model with basic features",
        "Need to add more feature engineering",
        "Consider ensemble methods"
    ]
})

wandb.finish()
```

### Step 2: Review Team Experiments
```python
# review_team_experiments.py
import wandb
import pandas as pd

# Get team experiments
api = wandb.Api()
runs = api.runs("team-ml-project")

# Create team results DataFrame
team_results = []
for run in runs:
    result = {
        "run_id": run.id,
        "run_name": run.name,
        "accuracy": run.summary.get("accuracy"),
        "tags": run.tags,
        "config": run.config
    }
    team_results.append(result)

df = pd.DataFrame(team_results)

# Display team results
print("Team Experiment Results:")
print(df[["run_name", "accuracy", "tags"]])

# Find best experiment
best_run = df.loc[df["accuracy"].idxmax()]
print(f"\nBest Experiment: {best_run['run_name']}")
print(f"Accuracy: {best_run['accuracy']:.4f}")

# Group by tags
if "sprint" in df.columns:
    sprint_results = df.groupby("tags")["accuracy"].agg(["mean", "max", "count"])
    print("\nSprint Results:")
    print(sprint_results)
```

## 📊 Monitoring and Debugging

### Check Experiment Status
```python
# Check experiment status
import wandb

api = wandb.Api()
run = api.run("project-name/run-id")

print(f"Run: {run.name}")
print(f"Status: {run.state}")
print(f"Metrics: {run.summary}")
print(f"Config: {run.config}")
```

### Monitor Model Performance
```python
# Monitor model performance
api = wandb.Api()
runs = api.runs("model-monitoring")

# Analyze performance trends
import pandas as pd
df = pd.DataFrame([run.summary for run in runs])
print(f"Average accuracy: {df['accuracy'].mean():.4f}")
print(f"Accuracy std: {df['accuracy'].std():.4f}")
```

## 🎯 Common Usage Patterns

### 1. **Data Science Workflow**
```python
# 1. Initialize wandb
wandb.init(project="data-science-workflow")

# 2. Log data and parameters
wandb.config.update({
    "dataset": "customer-data",
    "model_type": "random-forest"
})

# 3. Train and evaluate model
# (Training code here)

# 4. Log results
wandb.log({"accuracy": accuracy})
wandb.log_model(model, "final-model")

# 5. Finish run
wandb.finish()
```

### 2. **Model Development**
```python
# 1. Hyperparameter optimization
# (Use Sweeps)

# 2. Register best model
# (Use model registry)

# 3. Deploy model
# (Use deployment features)

# 4. Monitor model
# (Use monitoring features)
```

### 3. **Production Deployment**
```python
# 1. Load production model
api = wandb.Api()
model = api.artifact("project-name/model-name:latest")

# 2. Deploy to cloud
# (Use deployment features)

# 3. Monitor performance
wandb.init(project="production-monitoring")
wandb.log({"predictions": predictions, "ground_truth": ground_truth})
```

---

*WandB is now integrated into your ML workflow! 🎉*