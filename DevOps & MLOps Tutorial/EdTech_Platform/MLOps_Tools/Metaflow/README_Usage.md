# Metaflow — Usage

## 🚀 Getting Started with Metaflow

This guide covers practical usage examples for Metaflow in real-world ML projects.

## 📊 Example 1: Basic ML Flow

### Scenario: Simple Training Pipeline

Create a basic ML training workflow with Metaflow.

### Step 1: Define Flow
```python
# training_flow.py
from metaflow import FlowSpec, step, Parameter
import pandas as pd
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score
import joblib

class TrainingFlow(FlowSpec):
    """ML training workflow"""
    
    # Define parameters
    data_path = Parameter('data_path', default='data/train.csv')
    n_estimators = Parameter('n_estimators', default=100, type=int)
    test_size = Parameter('test_size', default=0.2, type=float)
    
    @step
    def start(self):
        """Load data"""
        self.data = pd.read_csv(self.data_path)
        print(f"Loaded {len(self.data)} samples")
        self.next(self.split)
    
    @step
    def split(self):
        """Split data"""
        X = self.data.drop('target', axis=1)
        y = self.data['target']
        
        self.X_train, self.X_test, self.y_train, self.y_test = \
            train_test_split(X, y, test_size=self.test_size, random_state=42)
        
        print(f"Train: {len(self.X_train)}, Test: {len(self.X_test)}")
        self.next(self.train)
    
    @step
    def train(self):
        """Train model"""
        self.model = RandomForestClassifier(
            n_estimators=self.n_estimators,
            random_state=42
        )
        self.model.fit(self.X_train, self.y_train)
        
        # Evaluate
        self.accuracy = accuracy_score(
            self.y_test,
            self.model.predict(self.X_test)
        )
        
        print(f"Model accuracy: {self.accuracy:.4f}")
        self.next(self.end)
    
    @step
    def end(self):
        """End step"""
        print("Training completed!")
        print(f"Final accuracy: {self.accuracy:.4f}")

if __name__ == '__main__':
    TrainingFlow()
```

### Step 2: Run Flow
```bash
# Run flow locally
python training_flow.py run

# Run with parameters
python training_flow.py run --n_estimators 200 --test_size 0.3

# View flow
python training_flow.py show
```

## 🔧 Example 2: Hyperparameter Tuning

### Scenario: Tune Model Hyperparameters

Use Metaflow to run multiple experiments with different hyperparameters.

### Step 1: Define Parameterized Flow
```python
# hyperparameter_flow.py
from metaflow import FlowSpec, step, Parameter
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import cross_val_score
import numpy as np

class HyperparameterFlow(FlowSpec):
    """Hyperparameter tuning workflow"""
    
    learning_rate = Parameter('learning_rate', default=0.01, type=float)
    n_estimators = Parameter('n_estimators', default=100, type=int)
    max_depth = Parameter('max_depth', default=10, type=int)
    
    @step
    def start(self):
        """Load data"""
        self.data = load_data()
        self.next(self.tune)
    
    @step
    def tune(self):
        """Tune hyperparameters"""
        model = RandomForestClassifier(
            n_estimators=self.n_estimators,
            max_depth=self.max_depth,
            random_state=42
        )
        
        # Cross-validation
        scores = cross_val_score(model, self.X, self.y, cv=5)
        self.mean_score = scores.mean()
        self.std_score = scores.std()
        
        print(f"Mean CV score: {self.mean_score:.4f} (+/- {self.std_score:.4f})")
        self.next(self.end)
    
    @step
    def end(self):
        """End step"""
        print(f"Best score: {self.mean_score:.4f}")

if __name__ == '__main__':
    HyperparameterFlow()
```

### Step 2: Run Multiple Experiments
```bash
# Run experiment 1
python hyperparameter_flow.py run --n_estimators 50 --max_depth 5

# Run experiment 2
python hyperparameter_flow.py run --n_estimators 100 --max_depth 10

# Run experiment 3
python hyperparameter_flow.py run --n_estimators 200 --max_depth 15

# Compare results
python hyperparameter_flow.py show
```

## 🚀 Example 3: Cloud Execution

### Scenario: Run Flow on AWS Batch

Execute Metaflow flows on AWS Batch for scalable execution.

### Step 1: Configure AWS
```bash
# Set AWS credentials
export AWS_DEFAULT_REGION=us-west-2
export METAFLOW_DATASTORE_SYSROOT_S3=s3://my-bucket/metaflow
```

### Step 2: Define Cloud Flow
```python
# cloud_flow.py
from metaflow import FlowSpec, step, batch

class CloudFlow(FlowSpec):
    @batch(cpu=4, memory=8000)
    @step
    def train(self):
        """Train model on cloud"""
        # Training code
        self.model = train_model()
        self.next(self.end)
    
    @step
    def end(self):
        """End step"""
        print("Cloud training completed")

if __name__ == '__main__':
    CloudFlow()
```

### Step 3: Run on Cloud
```bash
# Run on AWS Batch
python cloud_flow.py run --with batch

# Run on Kubernetes
python cloud_flow.py run --with kubernetes
```

## 🎯 Example 4: Parallel Processing

### Scenario: Process Multiple Data Sources in Parallel

Use Metaflow's foreach to process data in parallel.

### Step 1: Define Parallel Flow
```python
# parallel_flow.py
from metaflow import FlowSpec, step, foreach

class ParallelFlow(FlowSpec):
    @step
    def start(self):
        """Start step"""
        self.data_sources = ['source1.csv', 'source2.csv', 'source3.csv']
        self.next(self.process, foreach='data_sources')
    
    @step
    def process(self):
        """Process each data source"""
        data = load_data(self.input)
        self.processed = process_data(data)
        self.next(self.aggregate)
    
    @step
    def aggregate(self, inputs):
        """Aggregate results"""
        self.results = [inp.processed for inp in inputs]
        self.next(self.end)
    
    @step
    def end(self):
        """End step"""
        print(f"Aggregated {len(self.results)} results")

if __name__ == '__main__':
    ParallelFlow()
```

## 🔄 Example 5: Model Deployment

### Scenario: Deploy Model to Production

Use Metaflow to deploy trained models.

### Step 1: Define Deployment Flow
```python
# deployment_flow.py
from metaflow import FlowSpec, step

class DeploymentFlow(FlowSpec):
    @step
    def start(self):
        """Load model"""
        self.model = load_model()
        self.next(self.validate)
    
    @step
    def validate(self):
        """Validate model"""
        self.accuracy = validate_model(self.model)
        if self.accuracy < 0.8:
            raise ValueError("Model accuracy too low")
        self.next(self.deploy)
    
    @step
    def deploy(self):
        """Deploy model"""
        deploy_to_production(self.model)
        self.next(self.end)
    
    @step
    def end(self):
        """End step"""
        print("Model deployed successfully")

if __name__ == '__main__':
    DeploymentFlow()
```

## 📊 Monitoring and Debugging

### Check Flow Status
```bash
# List all runs
python my_flow.py runs

# Show specific run
python my_flow.py show <run-id>

# Resume failed run
python my_flow.py resume <run-id>
```

### Debug Flow Issues
```bash
# View flow logs
python my_flow.py logs <run-id> <step-name>

# Debug specific step
python my_flow.py resume <run-id> --step <step-name>
```

## 🎯 Common Usage Patterns

### 1. **Training Workflow**
```python
# 1. Define flow
class TrainingFlow(FlowSpec):
    @step
    def start(self):
        # Load data
        pass
    
    @step
    def train(self):
        # Train model
        pass

# 2. Run flow
# python training_flow.py run

# 3. View results
# python training_flow.py show
```

### 2. **Experiment Tracking**
```python
# Run multiple experiments
for lr in [0.01, 0.1, 1.0]:
    python my_flow.py run --learning_rate {lr}

# Compare results
python my_flow.py show
```

### 3. **Cloud Execution**
```python
# Run on cloud
python my_flow.py run --with batch

# Monitor execution
python my_flow.py show
```

---

*Metaflow is now integrated into your ML workflow! 🎉*