# Airflow — Usage

## 🚀 Getting Started with Airflow

This guide covers practical usage examples for Airflow in real-world ML and data pipeline projects.

## 📊 Example 1: Simple ML Pipeline

### Scenario: End-to-End ML Training Pipeline

Create a complete ML pipeline with data preparation, training, and evaluation.

### Step 1: Create DAG Structure
```python
# dags/ml_training_pipeline.py
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.bash import BashOperator
from datetime import datetime, timedelta
import pandas as pd
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score
import joblib

# Default arguments
default_args = {
    'owner': 'data-scientist',
    'depends_on_past': False,
    'start_date': datetime(2023, 1, 1),
    'email_on_failure': True,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5)
}

# Define DAG
dag = DAG(
    'ml_training_pipeline',
    default_args=default_args,
    description='End-to-end ML training pipeline',
    schedule_interval=timedelta(days=1),
    catchup=False,
    tags=['ml', 'training', 'daily']
)
```

### Step 2: Define Task Functions
```python
# Data preparation task
def prepare_data(**context):
    """Prepare and preprocess data"""
    # Load raw data
    df = pd.read_csv('/data/raw/train.csv')
    
    # Preprocess data
    df = df.dropna()
    df['feature_normalized'] = (df['feature'] - df['feature'].mean()) / df['feature'].std()
    
    # Split data
    train, test = train_test_split(df, test_size=0.2, random_state=42)
    
    # Save processed data
    train.to_csv('/data/processed/train.csv', index=False)
    test.to_csv('/data/processed/test.csv', index=False)
    
    print(f"Data prepared: {len(train)} train, {len(test)} test samples")
    
    # Return data info for next task
    return {
        'train_samples': len(train),
        'test_samples': len(test)
    }

# Model training task
def train_model(**context):
    """Train machine learning model"""
    # Get data info from previous task
    ti = context['ti']
    data_info = ti.xcom_pull(task_ids='prepare_data')
    
    # Load processed data
    train = pd.read_csv('/data/processed/train.csv')
    test = pd.read_csv('/data/processed/test.csv')
    
    # Prepare features and target
    X_train = train.drop('target', axis=1)
    y_train = train['target']
    X_test = test.drop('target', axis=1)
    y_test = test['target']
    
    # Train model
    model = RandomForestClassifier(n_estimators=100, random_state=42)
    model.fit(X_train, y_train)
    
    # Evaluate model
    accuracy = model.score(X_test, y_test)
    
    # Save model
    joblib.dump(model, '/models/model.pkl')
    
    print(f"Model trained - Accuracy: {accuracy:.4f}")
    
    # Return metrics
    return {
        'accuracy': accuracy,
        'model_path': '/models/model.pkl'
    }

# Model evaluation task
def evaluate_model(**context):
    """Evaluate model performance"""
    # Get model info from previous task
    ti = context['ti']
    model_info = ti.xcom_pull(task_ids='train_model')
    
    # Load model
    model = joblib.load(model_info['model_path'])
    
    # Load test data
    test = pd.read_csv('/data/processed/test.csv')
    X_test = test.drop('target', axis=1)
    y_test = test['target']
    
    # Make predictions
    predictions = model.predict(X_test)
    
    # Calculate metrics
    accuracy = accuracy_score(y_test, predictions)
    
    print(f"Model evaluation - Accuracy: {accuracy:.4f}")
    
    # Log metrics (could send to monitoring system)
    return {
        'accuracy': accuracy,
        'predictions_count': len(predictions)
    }
```

### Step 3: Create Tasks and Dependencies
```python
# Create tasks
prepare_task = PythonOperator(
    task_id='prepare_data',
    python_callable=prepare_data,
    dag=dag
)

train_task = PythonOperator(
    task_id='train_model',
    python_callable=train_model,
    dag=dag
)

evaluate_task = PythonOperator(
    task_id='evaluate_model',
    python_callable=evaluate_model,
    dag=dag
)

# Define task dependencies
prepare_task >> train_task >> evaluate_task
```

## 🔧 Example 2: Data Processing Pipeline with Sensors

### Scenario: Process Data When Available

Use sensors to wait for data files before processing.

### Step 1: Create Sensor-Based DAG
```python
# dags/data_processing_pipeline.py
from airflow import DAG
from airflow.sensors.filesystem import FileSensor
from airflow.operators.python import PythonOperator
from airflow.operators.bash import BashOperator
from datetime import datetime, timedelta

default_args = {
    'owner': 'data-engineer',
    'start_date': datetime(2023, 1, 1),
    'retries': 3,
    'retry_delay': timedelta(minutes=5)
}

dag = DAG(
    'data_processing_pipeline',
    default_args=default_args,
    description='Process data when files are available',
    schedule_interval='@hourly',
    catchup=False
)

# Wait for input file
wait_for_file = FileSensor(
    task_id='wait_for_input_file',
    filepath='/data/input/data.csv',
    poke_interval=60,  # Check every 60 seconds
    timeout=3600,  # Timeout after 1 hour
    dag=dag
)

# Process data
def process_data(**context):
    """Process input data"""
    import pandas as pd
    
    # Load data
    df = pd.read_csv('/data/input/data.csv')
    
    # Process data
    df_processed = df.dropna()
    df_processed['processed'] = True
    
    # Save processed data
    df_processed.to_csv('/data/processed/data.csv', index=False)
    
    print(f"Processed {len(df_processed)} records")

process_task = PythonOperator(
    task_id='process_data',
    python_callable=process_data,
    dag=dag
)

# Archive processed file
archive_task = BashOperator(
    task_id='archive_file',
    bash_command='mv /data/input/data.csv /data/archive/data_$(date +%Y%m%d_%H%M%S).csv',
    dag=dag
)

# Define dependencies
wait_for_file >> process_task >> archive_task
```

## 🚀 Example 3: Cloud Integration Pipeline

### Scenario: Process Data from S3 and Store Results

Use Airflow to orchestrate cloud-based data processing.

### Step 1: Create Cloud Pipeline
```python
# dags/cloud_data_pipeline.py
from airflow import DAG
from airflow.providers.amazon.aws.operators.s3 import S3FileTransformOperator
from airflow.providers.amazon.aws.sensors.s3 import S3KeySensor
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta

default_args = {
    'owner': 'cloud-engineer',
    'start_date': datetime(2023, 1, 1),
    'retries': 2,
    'retry_delay': timedelta(minutes=5)
}

dag = DAG(
    'cloud_data_pipeline',
    default_args=default_args,
    description='Process data from S3',
    schedule_interval='@daily',
    catchup=False
)

# Wait for S3 file
wait_for_s3_file = S3KeySensor(
    task_id='wait_for_s3_file',
    bucket_name='my-data-bucket',
    bucket_key='input/data.csv',
    aws_conn_id='aws_default',
    timeout=3600,
    poke_interval=60,
    dag=dag
)

# Download and process data
def process_s3_data(**context):
    """Download and process data from S3"""
    import boto3
    import pandas as pd
    
    # Create S3 client
    s3 = boto3.client('s3')
    
    # Download file
    s3.download_file('my-data-bucket', 'input/data.csv', '/tmp/data.csv')
    
    # Process data
    df = pd.read_csv('/tmp/data.csv')
    df_processed = df.dropna()
    
    # Save processed data
    df_processed.to_csv('/tmp/processed_data.csv', index=False)
    
    # Upload to S3
    s3.upload_file('/tmp/processed_data.csv', 'my-data-bucket', 'output/processed_data.csv')
    
    print(f"Processed and uploaded {len(df_processed)} records")

process_task = PythonOperator(
    task_id='process_s3_data',
    python_callable=process_s3_data,
    dag=dag
)

# Define dependencies
wait_for_s3_file >> process_task
```

## 🎯 Example 4: Parallel Processing Pipeline

### Scenario: Process Multiple Data Sources in Parallel

Use Airflow to process multiple data sources concurrently.

### Step 1: Create Parallel Processing DAG
```python
# dags/parallel_processing_pipeline.py
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta

default_args = {
    'owner': 'data-engineer',
    'start_date': datetime(2023, 1, 1),
    'retries': 1,
    'retry_delay': timedelta(minutes=5)
}

dag = DAG(
    'parallel_processing_pipeline',
    default_args=default_args,
    description='Process multiple data sources in parallel',
    schedule_interval='@daily',
    catchup=False
)

# Process different data sources
def process_source_a(**context):
    """Process data source A"""
    print("Processing data source A")
    return "source_a_complete"

def process_source_b(**context):
    """Process data source B"""
    print("Processing data source B")
    return "source_b_complete"

def process_source_c(**context):
    """Process data source C"""
    print("Processing data source C")
    return "source_c_complete"

# Aggregate results
def aggregate_results(**context):
    """Aggregate results from all sources"""
    ti = context['ti']
    
    # Get results from all parallel tasks
    result_a = ti.xcom_pull(task_ids='process_source_a')
    result_b = ti.xcom_pull(task_ids='process_source_b')
    result_c = ti.xcom_pull(task_ids='process_source_c')
    
    print(f"Aggregating results: {result_a}, {result_b}, {result_c}")
    return "aggregation_complete"

# Create parallel tasks
task_a = PythonOperator(
    task_id='process_source_a',
    python_callable=process_source_a,
    dag=dag
)

task_b = PythonOperator(
    task_id='process_source_b',
    python_callable=process_source_b,
    dag=dag
)

task_c = PythonOperator(
    task_id='process_source_c',
    python_callable=process_source_c,
    dag=dag
)

# Aggregate task
aggregate_task = PythonOperator(
    task_id='aggregate_results',
    python_callable=aggregate_results,
    dag=dag
)

# Define dependencies (parallel execution)
[task_a, task_b, task_c] >> aggregate_task
```

## 🔄 Example 5: Conditional Branching Pipeline

### Scenario: Conditional Processing Based on Data Quality

Use branching to handle different scenarios.

### Step 1: Create Conditional DAG
```python
# dags/conditional_pipeline.py
from airflow import DAG
from airflow.operators.python import PythonOperator, BranchPythonOperator
from airflow.operators.dummy import DummyOperator
from datetime import datetime, timedelta

default_args = {
    'owner': 'data-engineer',
    'start_date': datetime(2023, 1, 1),
    'retries': 1,
    'retry_delay': timedelta(minutes=5)
}

dag = DAG(
    'conditional_pipeline',
    default_args=default_args,
    description='Conditional processing based on data quality',
    schedule_interval='@daily',
    catchup=False
)

# Check data quality
def check_data_quality(**context):
    """Check data quality and return branch"""
    import pandas as pd
    
    # Load data
    df = pd.read_csv('/data/input/data.csv')
    
    # Check quality metrics
    null_percentage = df.isnull().sum().sum() / (len(df) * len(df.columns))
    duplicate_percentage = df.duplicated().sum() / len(df)
    
    # Determine branch based on quality
    if null_percentage < 0.1 and duplicate_percentage < 0.05:
        return 'process_high_quality_data'
    elif null_percentage < 0.3:
        return 'process_medium_quality_data'
    else:
        return 'reject_low_quality_data'

# Branch task
branch_task = BranchPythonOperator(
    task_id='check_data_quality',
    python_callable=check_data_quality,
    dag=dag
)

# Process high quality data
def process_high_quality(**context):
    """Process high quality data"""
    print("Processing high quality data")
    return "high_quality_processed"

high_quality_task = PythonOperator(
    task_id='process_high_quality_data',
    python_callable=process_high_quality,
    dag=dag
)

# Process medium quality data
def process_medium_quality(**context):
    """Process medium quality data with cleaning"""
    print("Processing medium quality data with cleaning")
    return "medium_quality_processed"

medium_quality_task = PythonOperator(
    task_id='process_medium_quality_data',
    python_callable=process_medium_quality,
    dag=dag
)

# Reject low quality data
reject_task = DummyOperator(
    task_id='reject_low_quality_data',
    dag=dag
)

# Final task
final_task = DummyOperator(
    task_id='final_task',
    trigger_rule='none_failed_or_skipped',
    dag=dag
)

# Define dependencies
branch_task >> [high_quality_task, medium_quality_task, reject_task]
[high_quality_task, medium_quality_task, reject_task] >> final_task
```

## 📊 Monitoring and Debugging

### Check DAG Status
```bash
# List all DAGs
airflow dags list

# Show DAG structure
airflow dags show ml_training_pipeline

# Check DAG for errors
airflow dags list-import-errors

# Test DAG
airflow dags test ml_training_pipeline 2023-01-01
```

### Monitor Task Execution
```bash
# List task instances
airflow tasks list ml_training_pipeline

# Test specific task
airflow tasks test ml_training_pipeline prepare_data 2023-01-01

# View task logs
airflow tasks logs ml_training_pipeline prepare_data 2023-01-01
```

### Debug DAG Issues
```bash
# Check DAG syntax
python dags/ml_training_pipeline.py

# Validate DAG
airflow dags list-import-errors

# View task dependencies
airflow dags show ml_training_pipeline --save output.png
```

## 🎯 Common Usage Patterns

### 1. **Daily ML Training**
```python
# Schedule daily model training
dag = DAG(
    'daily_model_training',
    schedule_interval='@daily',
    catchup=False
)

# Tasks: data preparation, training, evaluation, deployment
```

### 2. **Hourly Data Processing**
```python
# Schedule hourly data processing
dag = DAG(
    'hourly_data_processing',
    schedule_interval='@hourly',
    catchup=False
)

# Tasks: data extraction, transformation, loading
```

### 3. **Event-Driven Processing**
```python
# Trigger on file arrival
dag = DAG(
    'event_driven_processing',
    schedule_interval=None,  # No schedule
    catchup=False
)

# Use sensors to wait for events
```

---

*Airflow is now integrated into your workflow orchestration! 🎉*