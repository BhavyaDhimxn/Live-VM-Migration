# DVC — Usage

## 🚀 Getting Started with DVC

This guide covers practical usage examples for DVC in real-world ML projects.

## 📊 Example 1: Dataset Versioning

### Scenario: Version Control Training Data

Let's version control a dataset and track changes over time.

### Step 1: Prepare Your Dataset
```bash
# Create sample dataset
mkdir -p data/raw
echo "id,name,age,score" > data/raw/train.csv
echo "1,Alice,25,85" >> data/raw/train.csv
echo "2,Bob,30,92" >> data/raw/train.csv
echo "3,Charlie,28,78" >> data/raw/train.csv
```

### Step 2: Add Dataset to DVC
```bash
# Add dataset to DVC tracking
dvc add data/raw/train.csv

# This creates train.csv.dvc file
ls -la data/raw/
# train.csv.dvc  train.csv
```

### Step 3: Commit to Git
```bash
# Add DVC files to Git
git add data/raw/train.csv.dvc .gitignore
git commit -m "Add training dataset v1.0"
```

### Step 4: Update Dataset
```bash
# Add more data
echo "4,Diana,26,88" >> data/raw/train.csv
echo "5,Eve,29,91" >> data/raw/train.csv

# Update DVC tracking
dvc add data/raw/train.csv
git add data/raw/train.csv.dvc
git commit -m "Update training dataset v1.1"
```

### Step 5: Compare Versions
```bash
# Compare dataset versions
dvc diff HEAD~1 data/raw/train.csv
```

## 🔧 Example 2: ML Pipeline Management

### Scenario: End-to-End ML Pipeline

Create a complete ML pipeline with data preparation, training, and evaluation.

### Step 1: Create Pipeline Structure
```bash
# Create project structure
mkdir -p src data/{raw,processed} models metrics
```

### Step 2: Create Data Preparation Script
```python
# src/prepare.py
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
import yaml

def prepare_data():
    # Load data
    df = pd.read_csv('data/raw/train.csv')
    
    # Basic preprocessing
    df = df.dropna()
    df['age_normalized'] = (df['age'] - df['age'].mean()) / df['age'].std()
    
    # Split data
    train, test = train_test_split(df, test_size=0.2, random_state=42)
    
    # Save processed data
    train.to_csv('data/processed/train.csv', index=False)
    test.to_csv('data/processed/test.csv', index=False)
    
    # Save metrics
    metrics = {
        'train_size': len(train),
        'test_size': len(test),
        'features': len(df.columns) - 1
    }
    
    with open('metrics/prepare.json', 'w') as f:
        yaml.dump(metrics, f)
    
    print(f"Prepared data: {len(train)} train, {len(test)} test samples")

if __name__ == "__main__":
    prepare_data()
```

### Step 3: Create Training Script
```python
# src/train.py
import pandas as pd
import numpy as np
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error, r2_score
import joblib
import yaml

def train_model():
    # Load data
    train = pd.read_csv('data/processed/train.csv')
    test = pd.read_csv('data/processed/test.csv')
    
    # Prepare features and target
    X_train = train[['age', 'age_normalized']]
    y_train = train['score']
    X_test = test[['age', 'age_normalized']]
    y_test = test['score']
    
    # Train model
    model = LinearRegression()
    model.fit(X_train, y_train)
    
    # Evaluate model
    y_pred = model.predict(X_test)
    mse = mean_squared_error(y_test, y_pred)
    r2 = r2_score(y_test, y_pred)
    
    # Save model
    joblib.dump(model, 'models/model.pkl')
    
    # Save metrics
    metrics = {
        'mse': float(mse),
        'r2_score': float(r2),
        'train_samples': len(X_train),
        'test_samples': len(X_test)
    }
    
    with open('metrics/train.json', 'w') as f:
        yaml.dump(metrics, f)
    
    print(f"Model trained - MSE: {mse:.2f}, R²: {r2:.2f}")

if __name__ == "__main__":
    train_model()
```

### Step 4: Create Pipeline Definition
```yaml
# dvc.yaml
stages:
  prepare:
    cmd: python src/prepare.py
    deps:
    - src/prepare.py
    - data/raw/train.csv
    outs:
    - data/processed/train.csv
    - data/processed/test.csv
    metrics:
    - metrics/prepare.json

  train:
    cmd: python src/train.py
    deps:
    - src/train.py
    - data/processed/train.csv
    - data/processed/test.csv
    outs:
    - models/model.pkl
    metrics:
    - metrics/train.json
```

### Step 5: Run Pipeline
```bash
# Run entire pipeline
dvc repro

# Run specific stage
dvc repro prepare
dvc repro train

# Run with different parameters
dvc repro --set-param prepare.split_ratio=0.3
```

## 🔄 Example 3: Experiment Tracking

### Scenario: Hyperparameter Tuning

Track different experiments with various hyperparameters.

### Step 1: Create Parameters File
```yaml
# params.yaml
prepare:
  split_ratio: 0.8
  random_seed: 42

train:
  model_type: linear
  epochs: 100
  lr: 0.001
```

### Step 2: Run Experiments
```bash
# Baseline experiment
dvc exp run

# Experiment with different learning rate
dvc exp run --set-param train.lr=0.01

# Experiment with different model
dvc exp run --set-param train.model_type=random_forest

# Multiple parameter changes
dvc exp run --set-param train.lr=0.005 --set-param train.epochs=200
```

### Step 3: Compare Experiments
```bash
# List all experiments
dvc exp list

# Compare experiments
dvc exp diff

# Compare specific experiments
dvc exp diff HEAD~1 HEAD
```

### Step 4: Apply Best Experiment
```bash
# Apply best experiment
dvc exp apply <experiment_id>

# Or apply by name
dvc exp apply my-best-experiment
```

## ☁️ Example 4: Cloud Storage Integration

### Scenario: Working with AWS S3

Set up and use cloud storage for your ML project.

### Step 1: Configure S3 Remote
```bash
# Add S3 remote
dvc remote add -d myremote s3://my-ml-bucket/dvc-storage

# Configure credentials
export AWS_ACCESS_KEY_ID=your_access_key
export AWS_SECRET_ACCESS_KEY=your_secret_key
```

### Step 2: Push Data to Cloud
```bash
# Push data to S3
dvc push

# Push specific file
dvc push data/raw/train.csv.dvc
```

### Step 3: Pull Data from Cloud
```bash
# Pull all data
dvc pull

# Pull specific file
dvc pull data/raw/train.csv.dvc
```

### Step 4: Clone Project with Data
```bash
# Clone repository
git clone https://github.com/user/ml-project.git
cd ml-project

# Pull data from cloud
dvc pull
```

## 🔗 Integration Examples

### MLflow Integration
```python
# src/train_with_mlflow.py
import mlflow
import mlflow.sklearn
from sklearn.linear_model import LinearRegression

def train_with_mlflow():
    with mlflow.start_run():
        # Train model
        model = LinearRegression()
        model.fit(X_train, y_train)
        
        # Log model
        mlflow.sklearn.log_model(model, "model")
        
        # Log metrics
        mlflow.log_metric("mse", mse)
        mlflow.log_metric("r2", r2)
        
        # Log parameters
        mlflow.log_param("model_type", "linear")
        mlflow.log_param("epochs", 100)
```

### Git Integration
```bash
# Workflow with Git
git checkout -b feature/new-model
dvc repro
git add .
git commit -m "Add new model architecture"
git push origin feature/new-model
```

## 📊 Monitoring and Debugging

### Check Pipeline Status
```bash
# Check pipeline status
dvc status

# Check data status
dvc status --cloud

# Check metrics
dvc metrics show
```

### Debug Pipeline Issues
```bash
# Run with verbose output
dvc repro --verbose

# Check pipeline graph
dvc dag

# Check file dependencies
dvc dag --dot
```

## 🎯 Common Usage Patterns

### 1. **Data Science Workflow**
```bash
# 1. Add new dataset
dvc add data/new_dataset.csv
git add data/new_dataset.csv.dvc
git commit -m "Add new dataset"

# 2. Update pipeline
dvc repro

# 3. Compare results
dvc metrics diff
```

### 2. **Model Deployment**
```bash
# 1. Train model
dvc repro train

# 2. Tag model version
git tag v1.0
dvc push

# 3. Deploy specific version
git checkout v1.0
dvc pull
```

### 3. **Team Collaboration**
```bash
# 1. Pull latest changes
git pull
dvc pull

# 2. Make changes
dvc repro

# 3. Push changes
dvc push
git add .
git commit -m "Update model"
git push
```

---

*DVC is now integrated into your ML workflow! 🎉*