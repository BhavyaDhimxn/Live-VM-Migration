# Hopsworks — Usage

## 🚀 Getting Started with Hopsworks

This guide covers practical usage examples for Hopsworks in real-world ML projects.

## 📊 Example 1: Feature Store Operations

### Scenario: Create and Use Feature Groups

Let's create a feature group and use it for model training.

### Step 1: Connect to Hopsworks
```python
# connect_hopsworks.py
import hopsworks
import hsfs

# Connect to Hopsworks
project = hopsworks.login(
    api_key_value="your-api-key",
    project="my_project"
)

# Get feature store
connection = hsfs.connection()
fs = connection.get_feature_store("my_feature_store")
```

### Step 2: Create Feature Group
```python
# create_feature_group.py
import pandas as pd
from datetime import datetime

# Create sample data
data = pd.DataFrame({
    "driver_id": [1001, 1002, 1003],
    "avg_daily_trips": [10.5, 15.2, 8.3],
    "total_trips": [315, 456, 249],
    "event_timestamp": [datetime.now()] * 3
})

# Create feature group
fg = fs.create_feature_group(
    name="driver_features",
    version=1,
    description="Driver statistics features",
    primary_key=["driver_id"],
    event_time="event_timestamp"
)

# Insert data
fg.insert(data)
```

### Step 3: Read Features
```python
# read_features.py
# Read features from feature group
fg = fs.get_feature_group("driver_features", version=1)

# Read all features
features = fg.read()
print(features.head())

# Read with filters
features_filtered = fg.read(
    filter=fg.avg_daily_trips > 10
)
print(features_filtered.head())
```

## 🔧 Example 2: Model Training Pipeline

### Scenario: End-to-End ML Training Pipeline

Create a complete ML pipeline with feature engineering and model training.

### Step 1: Prepare Training Data
```python
# prepare_training_data.py
import pandas as pd
from sklearn.model_selection import train_test_split

# Get features from feature store
fg = fs.get_feature_group("driver_features", version=1)
features = fg.read()

# Prepare training data
X = features[["avg_daily_trips", "total_trips"]]
y = features["target"]  # Assuming target column exists

# Split data
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

print(f"Training samples: {len(X_train)}")
print(f"Test samples: {len(X_test)}")
```

### Step 2: Train Model
```python
# train_model.py
from sklearn.ensemble import RandomForestClassifier
import joblib

# Train model
model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)

# Evaluate model
accuracy = model.score(X_test, y_test)
print(f"Model accuracy: {accuracy:.4f}")

# Save model
joblib.dump(model, "model.pkl")
```

### Step 3: Register Model
```python
# register_model.py
import hsml

# Connect to model registry
mr = hsml.connection().get_model_registry()

# Create model
model = mr.python.create_model(
    name="driver_prediction_model",
    version=1,
    description="Driver prediction model"
)

# Save model
model.save("model.pkl")

# Log metrics
model.log_metric("accuracy", accuracy)
model.log_parameter("n_estimators", 100)
```

## 🚀 Example 3: Model Deployment

### Scenario: Deploy Model for Online Inference

Deploy a registered model for online serving.

### Step 1: Get Model from Registry
```python
# get_model.py
import hsml

# Get model from registry
mr = hsml.connection().get_model_registry()
model = mr.get_model("driver_prediction_model", version=1)

print(f"Model: {model.name}")
print(f"Version: {model.version}")
```

### Step 2: Create Serving Script
```python
# serving.py
import joblib
import pandas as pd

def predict(features):
    """Predict using deployed model"""
    # Load model
    model = joblib.load("model.pkl")
    
    # Make prediction
    prediction = model.predict(features)
    
    return prediction.tolist()
```

### Step 3: Deploy Model
```python
# deploy_model.py
# Deploy model for serving
deployment = model.deploy(
    name="driver_prediction_deployment",
    script_file="serving.py"
)

print(f"Deployment: {deployment.name}")
print(f"Status: {deployment.status}")
```

## 🎯 Example 4: ML Pipeline with Hopsworks

### Scenario: Complete ML Pipeline

Use Hopsworks pipelines for end-to-end ML workflows.

### Step 1: Define Pipeline
```python
# ml_pipeline.py
from hops import pipeline

@pipeline(name="training_pipeline")
def train_model():
    """Complete ML training pipeline"""
    import pandas as pd
    from sklearn.ensemble import RandomForestClassifier
    import joblib
    
    # Get features
    fg = fs.get_feature_group("driver_features", version=1)
    features = fg.read()
    
    # Prepare data
    X = features[["avg_daily_trips", "total_trips"]]
    y = features["target"]
    
    # Train model
    model = RandomForestClassifier(n_estimators=100, random_state=42)
    model.fit(X, y)
    
    # Evaluate
    accuracy = model.score(X, y)
    
    # Save model
    joblib.dump(model, "model.pkl")
    
    # Register model
    mr = hsml.connection().get_model_registry()
    model_reg = mr.python.create_model(
        name="driver_prediction_model",
        version=1
    )
    model_reg.save("model.pkl")
    model_reg.log_metric("accuracy", accuracy)
    
    return model
```

### Step 2: Run Pipeline
```python
# run_pipeline.py
from hops import pipeline

# Run pipeline
result = pipeline.run("training_pipeline")

print(f"Pipeline completed: {result}")
```

## 🔄 Example 5: Feature Engineering

### Scenario: Create Derived Features

Create new features from existing feature groups.

### Step 1: Read Base Features
```python
# feature_engineering.py
# Read base features
fg_base = fs.get_feature_group("driver_features", version=1)
features_base = fg_base.read()

print("Base features:")
print(features_base.head())
```

### Step 2: Create Derived Features
```python
# Create derived features
features_derived = features_base.copy()

# Create new features
features_derived["trips_per_week"] = features_derived["avg_daily_trips"] * 7
features_derived["efficiency_score"] = (
    features_derived["total_trips"] / features_derived["avg_daily_trips"]
)

print("Derived features:")
print(features_derived.head())
```

### Step 3: Save Derived Features
```python
# Create new feature group for derived features
fg_derived = fs.create_feature_group(
    name="driver_features_derived",
    version=1,
    description="Derived driver features",
    primary_key=["driver_id"],
    event_time="event_timestamp"
)

# Insert derived features
fg_derived.insert(features_derived)
```

## 📊 Monitoring and Debugging

### Check Feature Store Status
```python
# Check feature store
fs = connection.get_feature_store("my_feature_store")
print(f"Feature store: {fs.name}")

# List feature groups
feature_groups = fs.get_feature_groups()
print(f"Feature groups: {[fg.name for fg in feature_groups]}")
```

### Monitor Model Performance
```python
# Monitor model performance
mr = hsml.connection().get_model_registry()
model = mr.get_model("driver_prediction_model", version=1)

# Get metrics
metrics = model.get_metrics()
print(f"Model metrics: {metrics}")

# Get deployment status
deployment = model.get_deployment("driver_prediction_deployment")
print(f"Deployment status: {deployment.status}")
```

### Debug Pipeline Issues
```python
# Debug pipeline execution
from hops import pipeline

# Get pipeline status
status = pipeline.get_status("training_pipeline")
print(f"Pipeline status: {status}")

# Get pipeline logs
logs = pipeline.get_logs("training_pipeline")
print(f"Pipeline logs: {logs}")
```

## 🎯 Common Usage Patterns

### 1. **Feature Store Workflow**
```python
# 1. Create feature group
fg = fs.create_feature_group("my_features", version=1)

# 2. Insert features
fg.insert(features_df)

# 3. Read features
features = fg.read()

# 4. Use in training
model.fit(features[["feature1", "feature2"]], features["target"])
```

### 2. **Model Lifecycle Workflow**
```python
# 1. Train model
model = train_model()

# 2. Register model
model_reg = mr.python.create_model("my_model", version=1)
model_reg.save("model.pkl")

# 3. Deploy model
deployment = model_reg.deploy("production_deployment")

# 4. Monitor model
metrics = deployment.get_metrics()
```

### 3. **Pipeline Workflow**
```python
# 1. Define pipeline
@pipeline(name="ml_pipeline")
def ml_pipeline():
    # Pipeline steps
    pass

# 2. Run pipeline
pipeline.run("ml_pipeline")

# 3. Monitor pipeline
status = pipeline.get_status("ml_pipeline")
```

---

*Hopsworks is now integrated into your ML workflow! 🎉*