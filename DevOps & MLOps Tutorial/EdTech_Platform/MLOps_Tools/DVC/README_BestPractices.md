# DVC — Best Practices

## 🎯 Production Best Practices

### 1. **Project Structure**
```
ml-project/
├── .dvc/
├── .gitignore
├── dvc.yaml
├── params.yaml
├── data/
│   ├── raw/
│   ├── processed/
│   └── external/
├── src/
│   ├── prepare.py
│   ├── train.py
│   └── evaluate.py
├── models/
├── metrics/
├── notebooks/
└── tests/
```

### 2. **File Organization**
```bash
# Keep data files organized
data/
├── raw/           # Original, immutable data
├── processed/     # Cleaned, transformed data
├── external/      # External data sources
└── interim/       # Intermediate data files

# Separate code from data
src/
├── data/          # Data processing modules
├── models/        # Model definitions
├── features/      # Feature engineering
└── visualization/ # Plotting and visualization
```

## 🔐 Security Best Practices

### 1. **Credential Management**
```bash
# Use environment variables for credentials
export AWS_ACCESS_KEY_ID=your_access_key
export AWS_SECRET_ACCESS_KEY=your_secret_key

# Never commit credentials to Git
echo "*.local" >> .gitignore
echo ".env" >> .gitignore
```

### 2. **Local Configuration**
```bash
# Use .dvc/config.local for sensitive data
# This file is not committed to Git
dvc config --local remote.myremote.access_key_id YOUR_KEY
dvc config --local remote.myremote.secret_access_key YOUR_SECRET
```

### 3. **Access Control**
```bash
# Use IAM roles for cloud access
# Limit permissions to specific buckets
# Enable MFA for production environments
```

## 📊 Data Management Best Practices

### 1. **Data Versioning Strategy**
```bash
# Use semantic versioning for datasets
dvc add data/raw/train_v1.0.csv
dvc add data/raw/train_v1.1.csv

# Tag important dataset versions
git tag data-v1.0
git tag data-v1.1
```

### 2. **Data Quality Checks**
```python
# src/data_validation.py
import pandas as pd
import great_expectations as ge

def validate_data(df):
    # Check data types
    assert df.dtypes['age'] == 'int64'
    
    # Check value ranges
    assert df['age'].min() >= 0
    assert df['age'].max() <= 120
    
    # Check for missing values
    assert df.isnull().sum().sum() == 0
    
    # Use Great Expectations for comprehensive validation
    ge_df = ge.from_pandas(df)
    ge_df.expect_column_values_to_be_between('age', 0, 120)
    ge_df.save_expectation_suite('data/expectations.json')
```

### 3. **Data Lineage Tracking**
```yaml
# dvc.yaml with clear dependencies
stages:
  download:
    cmd: python src/download.py
    deps:
    - src/download.py
    outs:
    - data/raw/dataset.csv
    
  validate:
    cmd: python src/validate.py
    deps:
    - src/validate.py
    - data/raw/dataset.csv
    outs:
    - data/validated/dataset.csv
```

## 🔄 CI/CD Integration

### 1. **GitHub Actions Workflow**
```yaml
# .github/workflows/ml-pipeline.yml
name: ML Pipeline

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
    
    - name: Install DVC
      run: pip install dvc[s3]
    
    - name: Configure AWS
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      run: |
        dvc remote add -d myremote s3://my-bucket/dvc-storage
    
    - name: Pull data
      run: dvc pull
    
    - name: Run tests
      run: |
        dvc repro
        python -m pytest tests/
    
    - name: Push results
      run: dvc push
```

### 2. **Automated Model Training**
```bash
# Trigger training on data changes
# Use webhooks or scheduled jobs
dvc repro --force
dvc push
git add .
git commit -m "Auto-update model"
git push
```

## 📈 Model Monitoring Best Practices

### 1. **Model Versioning**
```bash
# Tag model versions
git tag model-v1.0
git tag model-v1.1

# Track model metadata
dvc metrics show models/model.pkl
```

### 2. **Performance Monitoring**
```python
# src/monitor.py
import mlflow
import pandas as pd
from sklearn.metrics import accuracy_score

def monitor_model_performance():
    # Load current model
    model = joblib.load('models/model.pkl')
    
    # Load test data
    test_data = pd.read_csv('data/test.csv')
    
    # Make predictions
    predictions = model.predict(test_data)
    
    # Calculate metrics
    accuracy = accuracy_score(test_data['target'], predictions)
    
    # Log to MLflow
    with mlflow.start_run():
        mlflow.log_metric("accuracy", accuracy)
        mlflow.log_metric("timestamp", time.time())
    
    # Alert if performance drops
    if accuracy < 0.8:
        send_alert(f"Model accuracy dropped to {accuracy}")
```

### 3. **Data Drift Detection**
```python
# src/drift_detection.py
from alibi_detect import TabularDrift

def detect_drift():
    # Load reference data
    reference_data = pd.read_csv('data/reference.csv')
    
    # Load new data
    new_data = pd.read_csv('data/new.csv')
    
    # Detect drift
    drift_detector = TabularDrift(reference_data.values)
    drift_score = drift_detector.score(new_data.values)
    
    if drift_score > 0.1:
        send_alert("Data drift detected!")
```

## ⚡ Performance Optimization

### 1. **Cache Configuration**
```bash
# Use hardlinks for better performance
dvc config cache.type hardlink

# Set appropriate cache size
dvc config cache.size 50GB

# Use SSD for cache directory
dvc config cache.dir /ssd/dvc-cache
```

### 2. **Parallel Processing**
```yaml
# dvc.yaml with parallel stages
stages:
  prepare:
    cmd: python src/prepare.py
    deps:
    - src/prepare.py
    - data/raw
    outs:
    - data/processed
    
  train:
    cmd: python src/train.py
    deps:
    - src/train.py
    - data/processed
    outs:
    - models/model.pkl
    # This stage runs after prepare completes
```

### 3. **Incremental Processing**
```python
# src/incremental_prepare.py
import os
from pathlib import Path

def incremental_prepare():
    # Check if output already exists
    if os.path.exists('data/processed/train.csv'):
        print("Processed data already exists, skipping...")
        return
    
    # Process data
    process_data()
```

## 🔧 Reproducibility Best Practices

### 1. **Environment Management**
```bash
# Use conda for environment management
conda create -n ml-project python=3.9
conda activate ml-project
conda env export > environment.yml

# Or use pip with requirements.txt
pip freeze > requirements.txt
```

### 2. **Random Seed Management**
```python
# src/config.py
import random
import numpy as np
import torch

def set_seeds(seed=42):
    random.seed(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)
    if torch.cuda.is_available():
        torch.cuda.manual_seed(seed)
```

### 3. **Parameter Management**
```yaml
# params.yaml with all hyperparameters
model:
  architecture: resnet50
  pretrained: true
  num_classes: 10

training:
  batch_size: 32
  learning_rate: 0.001
  epochs: 100
  optimizer: adam
  scheduler: cosine

data:
  image_size: 224
  augmentation: true
  validation_split: 0.2
```

## 🧪 Testing Best Practices

### 1. **Unit Tests**
```python
# tests/test_data_preparation.py
import pytest
import pandas as pd
from src.prepare import prepare_data

def test_data_preparation():
    # Test data preparation function
    prepare_data()
    
    # Check if output files exist
    assert os.path.exists('data/processed/train.csv')
    assert os.path.exists('data/processed/test.csv')
    
    # Check data quality
    train = pd.read_csv('data/processed/train.csv')
    assert len(train) > 0
    assert 'target' in train.columns
```

### 2. **Integration Tests**
```python
# tests/test_pipeline.py
def test_full_pipeline():
    # Run entire pipeline
    dvc repro
    
    # Check all outputs exist
    assert os.path.exists('models/model.pkl')
    assert os.path.exists('metrics/train.json')
    
    # Check model performance
    with open('metrics/train.json') as f:
        metrics = json.load(f)
    assert metrics['accuracy'] > 0.8
```

## 📚 Documentation Best Practices

### 1. **README Structure**
```markdown
# ML Project

## Setup
1. Clone repository
2. Install dependencies
3. Configure DVC
4. Pull data

## Usage
1. Run pipeline: `dvc repro`
2. View results: `dvc metrics show`
3. Compare experiments: `dvc exp diff`

## Data
- Raw data: `data/raw/`
- Processed data: `data/processed/`
- External data: `data/external/`
```

### 2. **Code Documentation**
```python
def prepare_data(input_path: str, output_path: str) -> None:
    """
    Prepare raw data for training.
    
    Args:
        input_path: Path to raw data file
        output_path: Path to save processed data
    
    Returns:
        None
    """
    # Implementation here
```

## 🚨 Common Pitfalls to Avoid

### 1. **Don't Commit Large Files**
```bash
# Always use .gitignore
echo "*.csv" >> .gitignore
echo "*.pkl" >> .gitignore
echo "*.h5" >> .gitignore
```

### 2. **Don't Hardcode Paths**
```python
# Bad
df = pd.read_csv('/home/user/data/train.csv')

# Good
df = pd.read_csv('data/raw/train.csv')
```

### 3. **Don't Ignore Data Quality**
```python
# Always validate data
def validate_data(df):
    assert not df.empty, "Data is empty"
    assert df.isnull().sum().sum() == 0, "Missing values found"
```

---

*Follow these best practices to build robust, scalable ML systems with DVC! 🎯*