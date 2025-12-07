# DataRobot — Usage

## 🚀 Getting Started with DataRobot

This guide covers practical usage examples for DataRobot in real-world ML projects.

## 📊 Example 1: Automated ML Project

### Scenario: Create and Run AutoML Project

Let's create an automated ML project with DataRobot.

### Step 1: Connect to DataRobot
```python
# connect_datarobot.py
import datarobot as dr

# Connect to DataRobot
client = dr.Client(
    token='your-api-token',
    endpoint='https://app.datarobot.com/api/v2'
)

print("Connected to DataRobot")
```

### Step 2: Create Project
```python
# create_project.py
import datarobot as dr
import pandas as pd

# Create sample data
data = pd.DataFrame({
    'feature1': [1, 2, 3, 4, 5],
    'feature2': [10, 20, 30, 40, 50],
    'target': [0, 1, 0, 1, 0]
})

# Save data
data.to_csv('data/train.csv', index=False)

# Create project
project = dr.Project.create(
    project_name='my-automl-project',
    sourcedata='data/train.csv'
)

print(f"Project created: {project.id}")
```

### Step 3: Start AutoML
```python
# start_automl.py
import datarobot as dr

# Get project
project = dr.Project.get(project_id)

# Set target and start autopilot
project.set_target(
    target='target',
    mode=dr.AUTOPILOT_MODE.FULL_AUTO
)

# Wait for autopilot to complete
project.wait_for_autopilot()

print("Autopilot completed!")
```

### Step 4: Get Best Model
```python
# get_best_model.py
import datarobot as dr

# Get project
project = dr.Project.get(project_id)

# Get all models
models = project.get_models()

# Sort by accuracy
models_sorted = sorted(
    models,
    key=lambda x: x.metrics['AUC'],
    reverse=True
)

# Get best model
best_model = models_sorted[0]

print(f"Best model: {best_model.model_type}")
print(f"AUC: {best_model.metrics['AUC']:.4f}")
```

## 🔧 Example 2: Model Deployment

### Scenario: Deploy Model for Production

Deploy the best model for production use.

### Step 1: Create Deployment
```python
# create_deployment.py
import datarobot as dr

# Get best model
project = dr.Project.get(project_id)
best_model = project.get_models()[0]

# Create deployment
deployment = dr.Deployment.create_from_learning_model(
    model_id=best_model.id,
    label='production-deployment',
    description='Production deployment for customer churn prediction'
)

print(f"Deployment created: {deployment.id}")
```

### Step 2: Make Predictions
```python
# make_predictions.py
import datarobot as dr
import pandas as pd

# Get deployment
deployment = dr.Deployment.get(deployment_id)

# Prepare prediction data
prediction_data = pd.DataFrame({
    'feature1': [6, 7, 8],
    'feature2': [60, 70, 80]
})

# Make predictions
predictions = deployment.predict(prediction_data)

print("Predictions:")
print(predictions)
```

## 🚀 Example 3: Model Monitoring

### Scenario: Monitor Deployed Model

Monitor model performance in production.

### Step 1: Get Monitoring Data
```python
# monitor_model.py
import datarobot as dr

# Get deployment
deployment = dr.Deployment.get(deployment_id)

# Get monitoring data
monitoring_data = deployment.get_association_ids()

print(f"Monitoring data: {monitoring_data}")
```

### Step 2: Check Model Performance
```python
# check_performance.py
import datarobot as dr

# Get deployment
deployment = dr.Deployment.get(deployment_id)

# Get performance metrics
metrics = deployment.get_performance_metrics()

print(f"Accuracy: {metrics.get('accuracy', 'N/A')}")
print(f"Latency: {metrics.get('latency', 'N/A')}")
```

## 🎯 Example 4: Batch Predictions

### Scenario: Make Batch Predictions

Use DataRobot for batch prediction processing.

### Step 1: Prepare Batch Data
```python
# batch_predictions.py
import datarobot as dr
import pandas as pd

# Prepare batch data
batch_data = pd.DataFrame({
    'feature1': range(100),
    'feature2': range(100, 200)
})

# Save batch data
batch_data.to_csv('data/batch_predictions.csv', index=False)
```

### Step 2: Run Batch Predictions
```python
# Run batch predictions
deployment = dr.Deployment.get(deployment_id)

# Create batch prediction job
job = dr.BatchPredictionJob.score(
    deployment_id=deployment.id,
    intake_settings={
        'type': 'local_file',
        'file': 'data/batch_predictions.csv'
    },
    output_settings={
        'type': 'local_file',
        'path': 'data/predictions.csv'
    }
)

# Wait for completion
job.wait_for_completion()

print("Batch predictions completed!")
```

## 🔄 Example 5: Model Retraining

### Scenario: Retrain Model with New Data

Retrain model with updated data.

### Step 1: Create New Project
```python
# retrain_model.py
import datarobot as dr

# Create new project with updated data
new_project = dr.Project.create(
    project_name='retrained-model',
    sourcedata='data/updated_train.csv'
)

# Set target
new_project.set_target(
    target='target',
    mode=dr.AUTOPILOT_MODE.FULL_AUTO
)

# Wait for completion
new_project.wait_for_autopilot()
```

### Step 2: Compare Models
```python
# Compare old and new models
old_model = dr.Model.get(old_model_id)
new_model = new_project.get_models()[0]

print(f"Old model AUC: {old_model.metrics['AUC']:.4f}")
print(f"New model AUC: {new_model.metrics['AUC']:.4f}")

# Deploy new model if better
if new_model.metrics['AUC'] > old_model.metrics['AUC']:
    deployment = dr.Deployment.create_from_learning_model(
        model_id=new_model.id,
        label='updated-production-deployment'
    )
    print("New model deployed!")
```

## 📊 Monitoring and Debugging

### Check Project Status
```python
# Check project status
project = dr.Project.get(project_id)
print(f"Project status: {project.status}")
print(f"Models trained: {len(project.get_models())}")
```

### Monitor Deployment
```python
# Monitor deployment
deployment = dr.Deployment.get(deployment_id)
print(f"Deployment status: {deployment.status}")
print(f"Model: {deployment.model_id}")
```

## 🎯 Common Usage Patterns

### 1. **AutoML Workflow**
```python
# 1. Create project
project = dr.Project.create(project_name='my-project', sourcedata='data.csv')

# 2. Start autopilot
project.set_target(target='target')
project.wait_for_autopilot()

# 3. Get best model
best_model = project.get_models()[0]

# 4. Deploy model
deployment = dr.Deployment.create_from_learning_model(model_id=best_model.id)
```

### 2. **Model Management**
```python
# 1. List all models
models = project.get_models()

# 2. Compare models
for model in models:
    print(f"{model.model_type}: {model.metrics['AUC']:.4f}")

# 3. Select best model
best_model = max(models, key=lambda x: x.metrics['AUC'])
```

### 3. **Production Deployment**
```python
# 1. Deploy model
deployment = dr.Deployment.create_from_learning_model(model_id=model.id)

# 2. Make predictions
predictions = deployment.predict(data)

# 3. Monitor performance
metrics = deployment.get_performance_metrics()
```

---

*DataRobot is now integrated into your ML workflow! 🎉*