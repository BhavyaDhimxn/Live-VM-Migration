# Feast — Usage

## 🚀 Getting Started with Feast

This guide covers practical usage examples for Feast in real-world ML projects.

## 📊 Example 1: Basic Feature Store Setup

### Scenario: Create and Use a Simple Feature Store

Let's create a feature store with driver statistics features.

### Step 1: Initialize Feature Store
```bash
# Create feature store repository
feast init driver_features
cd driver_features

# This creates:
# - feature_store.yaml
# - data/
# - example_repo.py
```

### Step 2: Define Features
```python
# driver_features.py
from feast import Entity, Feature, FeatureView, ValueType
from feast.data_source import FileSource
from datetime import timedelta
import pandas as pd

# Define entity
driver = Entity(
    name="driver_id",
    value_type=ValueType.INT64,
    description="Driver identifier"
)

# Create sample data
data = pd.DataFrame({
    "driver_id": [1001, 1002, 1003],
    "avg_daily_trips": [10.5, 15.2, 8.3],
    "total_trips": [315, 456, 249],
    "event_timestamp": pd.date_range("2023-01-01", periods=3, freq="D")
})

# Save data
data.to_parquet("data/driver_stats.parquet")

# Define feature view
driver_stats_fv = FeatureView(
    name="driver_stats",
    entities=["driver_id"],
    ttl=timedelta(days=1),
    features=[
        Feature(name="avg_daily_trips", dtype=ValueType.FLOAT),
        Feature(name="total_trips", dtype=ValueType.INT64),
    ],
    source=FileSource(
        path="data/driver_stats.parquet",
        timestamp_field="event_timestamp"
    )
)
```

### Step 3: Apply Feature Definitions
```bash
# Apply feature definitions
feast apply

# This creates the registry and validates features
```

### Step 4: Materialize Features
```bash
# Materialize features to online store
feast materialize-incremental $(date +%Y-%m-%d)

# Or materialize specific date range
feast materialize 2023-01-01 2023-01-31
```

### Step 5: Retrieve Online Features
```python
# get_features.py
from feast import FeatureStore
import pandas as pd

# Initialize feature store
fs = FeatureStore(repo_path=".")

# Get online features
features = fs.get_online_features(
    features=["driver_stats:avg_daily_trips", "driver_stats:total_trips"],
    entity_rows=[
        {"driver_id": 1001},
        {"driver_id": 1002},
        {"driver_id": 1003}
    ]
)

# Convert to DataFrame
df = features.to_df()
print(df)
```

## 🔧 Example 2: Training Data Preparation

### Scenario: Get Historical Features for Model Training

Use Feast to prepare training data with historical features.

### Step 1: Create Entity DataFrame
```python
# prepare_training_data.py
from feast import FeatureStore
import pandas as pd
from datetime import datetime, timedelta

# Initialize feature store
fs = FeatureStore(repo_path=".")

# Create entity DataFrame with timestamps
entity_df = pd.DataFrame({
    "driver_id": [1001, 1002, 1003, 1001, 1002, 1003],
    "event_timestamp": [
        datetime.now() - timedelta(days=i) 
        for i in range(6)
    ]
})

print("Entity DataFrame:")
print(entity_df)
```

### Step 2: Get Historical Features
```python
# Get historical features for training
training_df = fs.get_historical_features(
    entity_df=entity_df,
    features=[
        "driver_stats:avg_daily_trips",
        "driver_stats:total_trips"
    ]
)

# Convert to DataFrame
training_data = training_df.to_df()
print("\nTraining Data with Features:")
print(training_data)

# Save for model training
training_data.to_csv("data/training_data.csv", index=False)
```

### Step 3: Use in Model Training
```python
# train_model.py
import pandas as pd
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split

# Load training data
df = pd.read_csv("data/training_data.csv")

# Prepare features and target
X = df[["avg_daily_trips", "total_trips"]]
y = df["target"]  # Assuming target column exists

# Split data
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# Train model
model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)

# Evaluate
accuracy = model.score(X_test, y_test)
print(f"Model accuracy: {accuracy:.4f}")
```

## 🚀 Example 3: Online Feature Serving

### Scenario: Serve Features for Real-time Inference

Use Feast to serve features for online model inference.

### Step 1: Materialize Features
```bash
# Materialize latest features
feast materialize-incremental $(date +%Y-%m-%d)
```

### Step 2: Serve Features in API
```python
# feature_serving_api.py
from flask import Flask, request, jsonify
from feast import FeatureStore
import pandas as pd

app = Flask(__name__)
fs = FeatureStore(repo_path=".")

@app.route('/features', methods=['POST'])
def get_features():
    """Serve features for inference"""
    try:
        # Get entity IDs from request
        data = request.json
        driver_ids = data.get('driver_ids', [])
        
        # Prepare entity rows
        entity_rows = [{"driver_id": driver_id} for driver_id in driver_ids]
        
        # Get online features
        features = fs.get_online_features(
            features=[
                "driver_stats:avg_daily_trips",
                "driver_stats:total_trips"
            ],
            entity_rows=entity_rows
        )
        
        # Convert to dictionary
        result = features.to_dict()
        
        return jsonify(result)
    
    except Exception as e:
        return jsonify({"error": str(e)}), 500

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

### Step 3: Use in Model Inference
```python
# model_inference.py
import requests
import numpy as np
import joblib

# Load model
model = joblib.load("model.pkl")

# Get features from API
response = requests.post(
    "http://localhost:5000/features",
    json={"driver_ids": [1001, 1002]}
)

features = response.json()

# Prepare features for model
X = np.array([
    [
        features['avg_daily_trips'][i],
        features['total_trips'][i]
    ]
    for i in range(len(features['driver_id']))
])

# Make predictions
predictions = model.predict(X)
print(f"Predictions: {predictions}")
```

## 🎯 Example 4: Streaming Features

### Scenario: Process Streaming Data with Feast

Use Feast with streaming data sources like Kafka.

### Step 1: Define Streaming Source
```python
# streaming_features.py
from feast import Entity, Feature, FeatureView, ValueType
from feast.data_source import KafkaSource
from datetime import timedelta

# Define entity
driver = Entity(
    name="driver_id",
    value_type=ValueType.INT64
)

# Define streaming feature view
driver_stats_streaming_fv = FeatureView(
    name="driver_stats_streaming",
    entities=["driver_id"],
    ttl=timedelta(hours=1),
    features=[
        Feature(name="current_trip_count", dtype=ValueType.INT64),
        Feature(name="current_speed", dtype=ValueType.FLOAT),
    ],
    source=KafkaSource(
        kafka_bootstrap_servers="localhost:9092",
        topic="driver_stats",
        message_format="avro",
        timestamp_field="event_timestamp"
    ),
    stream=True  # Enable streaming
)
```

### Step 2: Process Streaming Data
```python
# process_streaming.py
from feast import FeatureStore
from kafka import KafkaConsumer
import json

fs = FeatureStore(repo_path=".")

# Create Kafka consumer
consumer = KafkaConsumer(
    'driver_stats',
    bootstrap_servers=['localhost:9092'],
    value_deserializer=lambda m: json.loads(m.decode('utf-8'))
)

# Process messages
for message in consumer:
    data = message.value
    
    # Update online store
    fs.write_to_online_store(
        feature_view="driver_stats_streaming",
        data=[data]
    )
    
    print(f"Updated features for driver {data['driver_id']}")
```

## 🔄 Example 5: Feature Versioning

### Scenario: Version Features and Track Changes

Use Feast to version features and track changes over time.

### Step 1: Create Feature Versions
```python
# feature_versions.py
from feast import Entity, Feature, FeatureView, ValueType
from feast.data_source import FileSource
from datetime import timedelta

# Version 1 of features
driver_stats_v1 = FeatureView(
    name="driver_stats_v1",
    entities=["driver_id"],
    ttl=timedelta(days=1),
    features=[
        Feature(name="avg_daily_trips", dtype=ValueType.FLOAT),
    ],
    source=FileSource(
        path="data/driver_stats_v1.parquet",
        timestamp_field="event_timestamp"
    ),
    tags={"version": "v1"}
)

# Version 2 with additional features
driver_stats_v2 = FeatureView(
    name="driver_stats_v2",
    entities=["driver_id"],
    ttl=timedelta(days=1),
    features=[
        Feature(name="avg_daily_trips", dtype=ValueType.FLOAT),
        Feature(name="total_trips", dtype=ValueType.INT64),  # New feature
        Feature(name="avg_rating", dtype=ValueType.FLOAT),    # New feature
    ],
    source=FileSource(
        path="data/driver_stats_v2.parquet",
        timestamp_field="event_timestamp"
    ),
    tags={"version": "v2"}
)
```

### Step 2: Apply and Use Versions
```bash
# Apply version 1
feast apply  # Applies driver_stats_v1

# Later, apply version 2
feast apply  # Applies driver_stats_v2

# Materialize both versions
feast materialize-incremental $(date +%Y-%m-%d)
```

### Step 3: Use Specific Versions
```python
# use_feature_versions.py
from feast import FeatureStore

fs = FeatureStore(repo_path=".")

# Use version 1 features
features_v1 = fs.get_online_features(
    features=["driver_stats_v1:avg_daily_trips"],
    entity_rows=[{"driver_id": 1001}]
)

# Use version 2 features
features_v2 = fs.get_online_features(
    features=[
        "driver_stats_v2:avg_daily_trips",
        "driver_stats_v2:total_trips",
        "driver_stats_v2:avg_rating"
    ],
    entity_rows=[{"driver_id": 1001}]
)

print("Version 1 features:", features_v1.to_dict())
print("Version 2 features:", features_v2.to_dict())
```

## 📊 Monitoring and Debugging

### Check Feature Store Status
```bash
# List feature views
feast feature-views list

# Describe feature view
feast feature-views describe driver_stats

# Check registry
feast registry-dump
```

### Monitor Feature Serving
```python
# Monitor feature serving performance
import time
from feast import FeatureStore

fs = FeatureStore(repo_path=".")

# Measure serving latency
start_time = time.time()
features = fs.get_online_features(
    features=["driver_stats:avg_daily_trips"],
    entity_rows=[{"driver_id": 1001}]
)
latency = time.time() - start_time

print(f"Feature serving latency: {latency*1000:.2f}ms")
```

### Debug Feature Issues
```python
# Debug feature retrieval
from feast import FeatureStore

fs = FeatureStore(repo_path=".")

# Check if features exist
try:
    features = fs.get_online_features(
        features=["driver_stats:avg_daily_trips"],
        entity_rows=[{"driver_id": 1001}]
    )
    print("Features retrieved successfully")
except Exception as e:
    print(f"Error retrieving features: {e}")
```

## 🎯 Common Usage Patterns

### 1. **Training Data Preparation**
```python
# 1. Create entity DataFrame
entity_df = pd.DataFrame({
    "driver_id": [1001, 1002, 1003],
    "event_timestamp": [datetime.now()] * 3
})

# 2. Get historical features
training_df = fs.get_historical_features(
    entity_df=entity_df,
    features=["driver_stats:avg_daily_trips"]
)

# 3. Use for training
training_data = training_df.to_df()
```

### 2. **Online Feature Serving**
```python
# 1. Materialize features
# feast materialize-incremental $(date +%Y-%m-%d)

# 2. Serve features
features = fs.get_online_features(
    features=["driver_stats:avg_daily_trips"],
    entity_rows=[{"driver_id": 1001}]
)

# 3. Use in inference
predictions = model.predict(features.to_df())
```

### 3. **Feature Updates**
```python
# 1. Update feature definitions
# (Edit feature views in Python)

# 2. Apply changes
# feast apply

# 3. Materialize updated features
# feast materialize-incremental $(date +%Y-%m-%d)
```

---

*Feast is now integrated into your ML workflow! 🎉*