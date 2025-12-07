# Airflow — Best Practices

## 🎯 Production Best Practices

### 1. **DAG Organization**
```
airflow/
├── dags/
│   ├── ml_pipelines/
│   │   ├── training_pipeline.py
│   │   ├── evaluation_pipeline.py
│   │   └── deployment_pipeline.py
│   ├── data_pipelines/
│   │   ├── etl_pipeline.py
│   │   └── data_quality_pipeline.py
│   └── utils/
│       ├── common_functions.py
│       └── helpers.py
├── plugins/
│   ├── operators/
│   ├── sensors/
│   └── hooks/
└── config/
    └── airflow.cfg
```

### 2. **DAG Naming Convention**
```python
# Consistent DAG naming
# Format: {domain}_{purpose}_{frequency}
# Examples:
# - ml_training_daily
# - data_processing_hourly
# - model_deployment_on_demand

dag = DAG(
    'ml_training_daily',
    description='Daily ML model training pipeline',
    schedule_interval='@daily',
    tags=['ml', 'training', 'daily']
)
```

## 🔐 Security Best Practices

### 1. **Secret Management**
```python
# Use Airflow Variables for secrets
from airflow.models import Variable

# Set secrets (via UI or CLI)
# airflow variables set api_key "secret-key"

# Get secrets in DAGs
api_key = Variable.get("api_key", deserialize_json=False)

# Use Connections for credentials
from airflow.models import Connection
from airflow.hooks.base import BaseHook

# Get connection
conn = BaseHook.get_connection("my_database")
username = conn.login
password = conn.password
```

### 2. **RBAC Configuration**
```python
# Enable RBAC
[webserver]
rbac = True

# Create custom roles
from airflow.www.security import AirflowSecurityManager

security_manager = AirflowSecurityManager()

# Create role with specific permissions
security_manager.create_role("DataScientist")
security_manager.add_permission("DataScientist", "can_read", "Dag")
security_manager.add_permission("DataScientist", "can_edit", "Dag")
security_manager.add_permission("DataScientist", "can_create", "Dag")
```

### 3. **Network Security**
```python
# Use VPN or private networks for database connections
[core]
sql_alchemy_conn = postgresql+psycopg2://user:password@private-db-host:5432/airflow

# Enable SSL for database connections
sql_alchemy_conn = postgresql+psycopg2://user:password@host:5432/airflow?sslmode=require
```

## 📊 Data Management Best Practices

### 1. **Idempotent Tasks**
```python
# Make tasks idempotent
def process_data(**context):
    """Idempotent data processing"""
    import pandas as pd
    from datetime import datetime
    
    # Use execution date for idempotency
    execution_date = context['execution_date']
    date_str = execution_date.strftime('%Y%m%d')
    
    # Check if already processed
    output_file = f'/data/processed/data_{date_str}.csv'
    if os.path.exists(output_file):
        print(f"Data already processed for {date_str}")
        return
    
    # Process data
    df = pd.read_csv('/data/input/data.csv')
    df_processed = df.dropna()
    df_processed.to_csv(output_file, index=False)
    
    print(f"Processed data for {date_str}")

task = PythonOperator(
    task_id='process_data',
    python_callable=process_data,
    dag=dag
)
```

### 2. **Data Quality Checks**
```python
# Implement data quality validation
def validate_data_quality(**context):
    """Validate data quality before processing"""
    import pandas as pd
    from great_expectations import dataset
    
    # Load data
    df = pd.read_csv('/data/input/data.csv')
    
    # Validate data
    ge_df = dataset.PandasDataset(df)
    
    # Check expectations
    validation_result = ge_df.validate_expectation_suite('data_expectations.json')
    
    if not validation_result.success:
        raise ValueError("Data quality validation failed")
    
    return "data_quality_passed"

validate_task = PythonOperator(
    task_id='validate_data_quality',
    python_callable=validate_data_quality,
    dag=dag
)
```

### 3. **Error Handling**
```python
# Implement proper error handling
from airflow.exceptions import AirflowException

def process_with_error_handling(**context):
    """Process data with error handling"""
    try:
        # Process data
        df = pd.read_csv('/data/input/data.csv')
        df_processed = df.dropna()
        df_processed.to_csv('/data/processed/data.csv', index=False)
        
        return "success"
    except FileNotFoundError as e:
        raise AirflowException(f"Input file not found: {e}")
    except Exception as e:
        # Log error and retry
        context['ti'].log.error(f"Error processing data: {e}")
        raise

task = PythonOperator(
    task_id='process_with_error_handling',
    python_callable=process_with_error_handling,
    on_failure_callback=send_failure_notification,
    dag=dag
)
```

## 🔄 CI/CD Integration

### 1. **DAG Testing**
```python
# test_dags.py
import pytest
from airflow.models import DagBag

def test_dag_loaded():
    """Test that DAGs are loaded correctly"""
    dag_bag = DagBag()
    assert len(dag_bag.dags) > 0
    
    for dag_id, dag in dag_bag.dags.items():
        assert dag is not None
        assert len(dag.tasks) > 0

def test_dag_structure():
    """Test DAG structure"""
    dag_bag = DagBag()
    dag = dag_bag.get_dag('ml_training_pipeline')
    
    assert dag is not None
    assert 'prepare_data' in [task.task_id for task in dag.tasks]
    assert 'train_model' in [task.task_id for task in dag.tasks]
```

### 2. **Automated Deployment**
```yaml
# .github/workflows/airflow-deploy.yml
name: Deploy Airflow DAGs

on:
  push:
    branches: [main]
    paths:
      - 'dags/**'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    
    - name: Test DAGs
      run: |
        pip install apache-airflow
        python -m pytest tests/test_dags.py
    
    - name: Deploy to Airflow
      run: |
        # Copy DAGs to Airflow server
        scp -r dags/ user@airflow-server:/home/user/airflow/dags/
```

## 📈 Monitoring Best Practices

### 1. **Task Monitoring**
```python
# Implement task monitoring
from airflow.operators.python import PythonOperator
from airflow.utils.email import send_email

def monitor_task_performance(**context):
    """Monitor task performance"""
    import time
    
    start_time = time.time()
    
    # Execute task
    result = execute_task()
    
    execution_time = time.time() - start_time
    
    # Log metrics
    context['ti'].xcom_push(key='execution_time', value=execution_time)
    context['ti'].xcom_push(key='result', value=result)
    
    # Alert if slow
    if execution_time > 300:  # 5 minutes
        send_email(
            to=['team@example.com'],
            subject='Slow Task Alert',
            html_content=f'Task took {execution_time} seconds'
        )
    
    return result

task = PythonOperator(
    task_id='monitored_task',
    python_callable=monitor_task_performance,
    dag=dag
)
```

### 2. **DAG Performance Monitoring**
```python
# Monitor DAG execution
from airflow.models import DagRun
from airflow.utils.state import State

def check_dag_performance():
    """Check DAG performance metrics"""
    dag_runs = DagRun.find(dag_id='ml_training_pipeline')
    
    success_count = sum(1 for run in dag_runs if run.state == State.SUCCESS)
    failure_count = sum(1 for run in dag_runs if run.state == State.FAILED)
    
    success_rate = success_count / len(dag_runs) if dag_runs else 0
    
    if success_rate < 0.9:  # 90% success rate
        send_alert(f"DAG success rate is {success_rate:.2%}")
    
    return success_rate
```

### 3. **Resource Monitoring**
```python
# Monitor resource usage
import psutil
import time

def monitor_resources(**context):
    """Monitor system resources"""
    cpu_percent = psutil.cpu_percent(interval=1)
    memory_percent = psutil.virtual_memory().percent
    disk_percent = psutil.disk_usage('/').percent
    
    # Log metrics
    context['ti'].xcom_push(key='cpu_percent', value=cpu_percent)
    context['ti'].xcom_push(key='memory_percent', value=memory_percent)
    context['ti'].xcom_push(key='disk_percent', value=disk_percent)
    
    # Alert if resources are high
    if cpu_percent > 80 or memory_percent > 80:
        send_alert(f"High resource usage: CPU {cpu_percent}%, Memory {memory_percent}%")
```

## ⚡ Performance Optimization

### 1. **Task Optimization**
```python
# Optimize task execution
from airflow.operators.python import PythonOperator

# Use task pools for resource management
task = PythonOperator(
    task_id='resource_intensive_task',
    python_callable=process_data,
    pool='high_memory_pool',  # Limit concurrent executions
    pool_slots=2,
    dag=dag
)

# Use task timeouts
task = PythonOperator(
    task_id='timeout_task',
    python_callable=process_data,
    execution_timeout=timedelta(minutes=30),
    dag=dag
)
```

### 2. **DAG Optimization**
```python
# Optimize DAG structure
dag = DAG(
    'optimized_pipeline',
    default_args=default_args,
    max_active_runs=1,  # Limit concurrent runs
    max_active_tasks=10,  # Limit concurrent tasks
    dagrun_timeout=timedelta(hours=2),  # DAG timeout
    catchup=False  # Don't backfill
)
```

### 3. **Database Optimization**
```ini
# Optimize database connections
[core]
sql_alchemy_pool_size = 5
sql_alchemy_max_overflow = 10
sql_alchemy_pool_recycle = 1800
sql_alchemy_pool_pre_ping = True
```

## 🔧 Reproducibility Best Practices

### 1. **Version Control**
```bash
# Store DAGs in Git
git init
git add dags/
git commit -m "Add ML training pipeline"

# Use tags for versions
git tag v1.0.0
```

### 2. **Environment Management**
```python
# Use consistent Python versions
# requirements.txt
apache-airflow==2.7.0
pandas==1.5.3
scikit-learn==1.2.0

# Use virtual environments
python -m venv airflow-env
source airflow-env/bin/activate
pip install -r requirements.txt
```

### 3. **Configuration Management**
```python
# Use environment-specific configurations
import os

# Development
if os.getenv('ENVIRONMENT') == 'development':
    dag = DAG(
        'ml_pipeline',
        schedule_interval='@daily',
        catchup=False
    )
# Production
elif os.getenv('ENVIRONMENT') == 'production':
    dag = DAG(
        'ml_pipeline',
        schedule_interval='@daily',
        catchup=False,
        max_active_runs=1
    )
```

## 🧪 Testing Best Practices

### 1. **Unit Tests**
```python
# test_tasks.py
import pytest
from airflow.models import DagBag

def test_task_functions():
    """Test task functions"""
    from dags.ml_pipeline import prepare_data, train_model
    
    # Test prepare_data
    result = prepare_data()
    assert result is not None
    assert 'train_samples' in result
    
    # Test train_model
    result = train_model()
    assert result is not None
    assert 'accuracy' in result
```

### 2. **Integration Tests**
```python
# test_dag_integration.py
import pytest
from airflow.models import DagBag, TaskInstance
from datetime import datetime

def test_dag_execution():
    """Test DAG execution"""
    dag_bag = DagBag()
    dag = dag_bag.get_dag('ml_training_pipeline')
    
    # Create test execution
    execution_date = datetime(2023, 1, 1)
    dag_run = dag.create_dagrun(
        run_id='test_run',
        execution_date=execution_date,
        state='running'
    )
    
    # Execute tasks
    for task in dag.tasks:
        ti = TaskInstance(task, execution_date)
        ti.run()
        assert ti.state == 'success'
```

## 📚 Documentation Best Practices

### 1. **DAG Documentation**
```python
dag = DAG(
    'ml_training_pipeline',
    description="""
    Daily ML model training pipeline.
    
    This pipeline:
    1. Prepares and preprocesses data
    2. Trains a machine learning model
    3. Evaluates model performance
    4. Deploys model if performance is acceptable
    
    Schedule: Daily at midnight UTC
    Owner: ML Team
    """,
    doc_md="""
    # ML Training Pipeline
    
    ## Overview
    This DAG trains a machine learning model daily.
    
    ## Tasks
    - prepare_data: Prepares and preprocesses training data
    - train_model: Trains the ML model
    - evaluate_model: Evaluates model performance
    """,
    tags=['ml', 'training', 'daily']
)
```

### 2. **Task Documentation**
```python
def prepare_data(**context):
    """
    Prepare and preprocess data for model training.
    
    Args:
        context: Airflow context dictionary
    
    Returns:
        dict: Dictionary with train and test sample counts
    
    Raises:
        FileNotFoundError: If input data file is not found
    """
    # Implementation
    pass

task = PythonOperator(
    task_id='prepare_data',
    python_callable=prepare_data,
    doc_md="""
    ## Prepare Data Task
    
    This task prepares and preprocesses data for model training.
    It handles missing values, feature normalization, and data splitting.
    """,
    dag=dag
)
```

## 🚨 Common Pitfalls to Avoid

### 1. **Don't Use Dynamic Start Dates**
```python
# Bad: Dynamic start date
dag = DAG(
    'bad_dag',
    start_date=datetime.now()  # Changes every time
)

# Good: Fixed start date
dag = DAG(
    'good_dag',
    start_date=datetime(2023, 1, 1)  # Fixed date
)
```

### 2. **Don't Ignore Task Dependencies**
```python
# Bad: No explicit dependencies
task1 = PythonOperator(...)
task2 = PythonOperator(...)
# Missing: task1 >> task2

# Good: Explicit dependencies
task1 = PythonOperator(...)
task2 = PythonOperator(...)
task1 >> task2
```

### 3. **Don't Hardcode Values**
```python
# Bad: Hardcoded values
def process_data():
    df = pd.read_csv('/hardcoded/path/data.csv')

# Good: Use variables
from airflow.models import Variable

def process_data():
    data_path = Variable.get("data_path")
    df = pd.read_csv(data_path)
```

---

*Follow these best practices to build robust, scalable workflows with Airflow! 🎯*