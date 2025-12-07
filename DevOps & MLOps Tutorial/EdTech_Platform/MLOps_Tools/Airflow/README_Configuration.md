# Airflow — Configuration

## ⚙️ Configuration Overview

Airflow configuration is managed through the `airflow.cfg` file and environment variables, allowing you to customize behavior for different environments and use cases.

## 📁 Configuration Files

### 1. **Airflow Configuration** (`airflow.cfg`)
```ini
[core]
# Airflow home directory
airflow_home = /home/user/airflow

# DAGs folder
dags_folder = /home/user/airflow/dags

# Executor
executor = LocalExecutor

# Timezone
default_timezone = UTC

# Database connection
sql_alchemy_conn = postgresql+psycopg2://user:password@localhost/airflow

# Parallelism
parallelism = 32
dag_concurrency = 16
max_active_runs_per_dag = 16

[webserver]
# Web server host and port
web_server_host = 0.0.0.0
web_server_port = 8080

# Authentication
auth_backend = airflow.contrib.auth.backends.password_auth

# Secret key
secret_key = your-secret-key-here

[scheduler]
# Scheduler settings
job_heartbeat_sec = 5
scheduler_heartbeat_sec = 5

# DAG processing
dag_dir_list_interval = 300
max_threads = 2
```

### 2. **Environment Variables**
```bash
# Set Airflow home
export AIRFLOW_HOME=~/airflow

# Set database connection
export AIRFLOW__CORE__SQL_ALCHEMY_CONN="postgresql+psycopg2://user:password@localhost/airflow"

# Set executor
export AIRFLOW__CORE__EXECUTOR="LocalExecutor"

# Set parallelism
export AIRFLOW__CORE__PARALLELISM=32
export AIRFLOW__CORE__DAG_CONCURRENCY=16
```

## 🔧 Core Configuration

### 1. **Database Configuration**
```ini
# PostgreSQL
[core]
sql_alchemy_conn = postgresql+psycopg2://user:password@localhost:5432/airflow

# MySQL
[core]
sql_alchemy_conn = mysql+pymysql://user:password@localhost:3306/airflow

# SQLite (development only)
[core]
sql_alchemy_conn = sqlite:////home/user/airflow/airflow.db
```

### 2. **Executor Configuration**
```ini
# SequentialExecutor (default, single task)
[core]
executor = SequentialExecutor

# LocalExecutor (multiple tasks, same machine)
[core]
executor = LocalExecutor

# CeleryExecutor (distributed)
[core]
executor = CeleryExecutor
celery_broker_url = redis://localhost:6379/0
celery_result_backend = db+postgresql://user:password@localhost/airflow

# KubernetesExecutor
[core]
executor = KubernetesExecutor
kubernetes_namespace = airflow
```

### 3. **DAG Configuration**
```ini
[core]
# DAGs folder
dags_folder = /home/user/airflow/dags

# DAG processing interval
dag_dir_list_interval = 300

# Maximum concurrent DAG runs
max_active_runs_per_dag = 16

# DAG concurrency
dag_concurrency = 16

# Task concurrency
task_concurrency = 16
```

## ☁️ Cloud Provider Configuration

### 1. **AWS Configuration**
```ini
# AWS connection
[aws]
aws_access_key_id = your-access-key
aws_secret_access_key = your-secret-key
aws_default_region = us-west-2

# S3 connection
[connections]
conn_id = aws_s3
conn_type = aws
extra = {"region_name": "us-west-2"}
```

### 2. **Google Cloud Configuration**
```ini
# GCP connection
[gcp]
project_id = your-project-id
key_path = /path/to/service-account-key.json

# BigQuery connection
[connections]
conn_id = bigquery_default
conn_type = google_cloud_platform
extra = {"project": "your-project-id"}
```

### 3. **Azure Configuration**
```ini
# Azure connection
[azure]
tenant_id = your-tenant-id
client_id = your-client-id
client_secret = your-client-secret

# Azure Blob connection
[connections]
conn_id = azure_blob_default
conn_type = wasb
extra = {"account_name": "your-account", "account_key": "your-key"}
```

## 🔐 Security Configuration

### 1. **Authentication Configuration**
```ini
# Password authentication
[webserver]
auth_backend = airflow.contrib.auth.backends.password_auth

# LDAP authentication
[webserver]
auth_backend = airflow.contrib.auth.backends.ldap_auth
[ldap]
uri = ldap://ldap.example.com
bind_user = cn=admin,dc=example,dc=com
bind_password = password
user_filter = (uid={username})
user_name_attr = uid
group_member_attr = memberOf
superuser_filter = (memberOf=cn=airflow-superusers,ou=groups,dc=example,dc=com)
```

### 2. **RBAC Configuration**
```python
# Enable RBAC
[webserver]
rbac = True

# Create roles and permissions
from airflow.www.security import AirflowSecurityManager

security_manager = AirflowSecurityManager()
security_manager.create_role("DataScientist")
security_manager.add_permission("DataScientist", "can_read", "Dag")
security_manager.add_permission("DataScientist", "can_edit", "Dag")
```

### 3. **Secret Management**
```python
# Use Airflow Variables for secrets
from airflow.models import Variable

# Set secret
Variable.set("api_key", "secret-key", serialize_json=False)

# Get secret
api_key = Variable.get("api_key")

# Use Connections for credentials
from airflow.models import Connection

conn = Connection(
    conn_id="my_conn",
    conn_type="http",
    host="api.example.com",
    login="username",
    password="password"
)
```

## 🔄 Advanced Configuration

### 1. **Email Configuration**
```ini
[email]
email_backend = airflow.providers.smtp.hooks.smtp.SmtpHook
smtp_host = smtp.gmail.com
smtp_starttls = True
smtp_ssl = False
smtp_user = your-email@gmail.com
smtp_password = your-password
smtp_port = 587
smtp_mail_from = airflow@example.com
```

### 2. **Logging Configuration**
```ini
[logging]
# Log level
logging_level = INFO

# Log format
log_format = [%%(asctime)s] {%%(filename)s:%%(lineno)d} %%(levelname)s - %%(message)s
simple_log_format = %%(asctime)s %%(levelname)s - %%(message)s

# Log file location
base_log_folder = /home/user/airflow/logs
```

### 3. **Monitoring Configuration**
```ini
# StatsD configuration
[statsd]
statsd_on = True
statsd_host = localhost
statsd_port = 8125
statsd_prefix = airflow

# Sentry configuration
[sentry]
sentry_dsn = https://your-sentry-dsn@sentry.io/project-id
```

## 🐛 Common Configuration Issues

### Issue 1: Database Connection Failed
```bash
# Error: Cannot connect to database
# Solution: Check database configuration
# Test PostgreSQL connection
psql -h localhost -U airflow -d airflow

# Test MySQL connection
mysql -h localhost -u airflow -p airflow
```

### Issue 2: Executor Not Working
```bash
# Error: Executor not configured correctly
# Solution: Check executor configuration
airflow config get-value core executor

# For CeleryExecutor, check broker
airflow config get-value celery broker_url
```

### Issue 3: DAGs Not Appearing
```bash
# Error: DAGs not showing in UI
# Solution: Check DAGs folder
airflow config get-value core dags_folder

# Check DAG processing
airflow dags list-import-errors
```

### Issue 4: Authentication Issues
```bash
# Error: Cannot login to web UI
# Solution: Reset admin user
airflow users create \
    --username admin \
    --firstname Admin \
    --lastname User \
    --role Admin \
    --email admin@example.com \
    --password admin
```

## ✅ Configuration Validation

### Test Configuration
```bash
# Validate configuration
airflow config list

# Check specific setting
airflow config get-value core executor

# Test database connection
airflow db check

# Test DAGs
airflow dags list
```

### Configuration Checklist
- [ ] Database connection configured and accessible
- [ ] Executor configured correctly
- [ ] DAGs folder set and accessible
- [ ] Authentication configured
- [ ] Email configuration set (if needed)
- [ ] Cloud provider connections configured (if needed)
- [ ] Logging configured
- [ ] Monitoring configured (if needed)

## 🔧 Configuration Best Practices

1. **🔐 Security**: Use environment variables for sensitive data
2. **💾 Database**: Use PostgreSQL or MySQL for production
3. **🔄 Executor**: Choose appropriate executor for your workload
4. **📊 Monitoring**: Enable monitoring and alerting
5. **📝 Documentation**: Document all configuration changes
6. **🧪 Testing**: Test configuration in development first

---

*Your Airflow configuration is now optimized for your workflow orchestration! 🎯*