# Prometheus — Usage

## 🚀 Getting Started with Prometheus

This guide covers practical usage examples for Prometheus in real-world monitoring scenarios.

## 📊 Example 1: Basic Metrics Collection

### Scenario: Monitor Application Metrics

Set up Prometheus to collect metrics from a web application.

### Step 1: Configure Prometheus
```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'web-app'
    static_configs:
      - targets: ['localhost:8080']
        labels:
          environment: 'production'
          service: 'web-app'
```

### Step 2: Instrument Application
```python
# app.py - Python application with Prometheus metrics
from prometheus_client import start_http_server, Counter, Histogram
import time

# Define metrics
REQUEST_COUNT = Counter('http_requests_total', 'Total HTTP requests', ['method', 'endpoint'])
REQUEST_DURATION = Histogram('http_request_duration_seconds', 'HTTP request duration')

# Start metrics server
start_http_server(8000)

# Use metrics in application
@REQUEST_DURATION.time()
def handle_request(method, endpoint):
    REQUEST_COUNT.labels(method=method, endpoint=endpoint).inc()
    # Handle request
    pass
```

### Step 3: Query Metrics
```promql
# Query total requests
http_requests_total

# Query requests by method
http_requests_total{method="GET"}

# Calculate request rate
rate(http_requests_total[5m])

# Calculate error rate
rate(http_requests_total{status=~"5.."}[5m])
```

## 🔧 Example 2: System Metrics Monitoring

### Scenario: Monitor Server Infrastructure

Monitor CPU, memory, and disk usage using Node Exporter.

### Step 1: Install Node Exporter
```bash
# Run Node Exporter
docker run -d \
  --name node-exporter \
  -p 9100:9100 \
  prom/node-exporter
```

### Step 2: Configure Prometheus
```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
        labels:
          instance: 'server-1'
```

### Step 3: Query System Metrics
```promql
# CPU usage
100 - (avg(irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Disk usage
100 - ((node_filesystem_avail_bytes{mountpoint="/"} * 100) / node_filesystem_size_bytes{mountpoint="/"})
```

## 🚀 Example 3: Alert Configuration

### Scenario: Set Up Alerts for Critical Metrics

Configure alerts to notify when metrics exceed thresholds.

### Step 1: Create Alert Rules
```yaml
# alerts.yml
groups:
  - name: infrastructure
    interval: 30s
    rules:
      - alert: HighCPUUsage
        expr: 100 - (avg(irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage detected"
          description: "CPU usage is above 80% for 5 minutes"
      
      - alert: HighMemoryUsage
        expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 90
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High memory usage detected"
          description: "Memory usage is above 90%"
```

### Step 2: Configure Alertmanager
```yaml
# alertmanager.yml
route:
  group_by: ['alertname', 'cluster']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  receiver: 'default-receiver'
  routes:
    - match:
        severity: critical
      receiver: 'critical-receiver'

receivers:
  - name: 'default-receiver'
    email_configs:
      - to: 'team@example.com'
        subject: 'Prometheus Alert'
  
  - name: 'critical-receiver'
    email_configs:
      - to: 'oncall@example.com'
        subject: 'CRITICAL: Prometheus Alert'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL'
        channel: '#alerts'
```

### Step 3: Start Alertmanager
```bash
# Run Alertmanager
docker run -d \
  --name alertmanager \
  -p 9093:9093 \
  -v $(pwd)/alertmanager.yml:/etc/alertmanager/alertmanager.yml \
  prom/alertmanager:latest
```

## 🎯 Example 4: Kubernetes Service Discovery

### Scenario: Auto-discover Pods in Kubernetes

Use Prometheus to automatically discover and scrape metrics from Kubernetes pods.

### Step 1: Configure Kubernetes Service Discovery
```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
```

### Step 2: Annotate Pods
```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "8080"
    prometheus.io/path: "/metrics"
spec:
  containers:
  - name: app
    image: my-app:latest
```

## 🔄 Example 5: Recording Rules

### Scenario: Pre-compute Frequently Used Queries

Create recording rules to pre-compute expensive queries.

### Step 1: Define Recording Rules
```yaml
# recording-rules.yml
groups:
  - name: recording_rules
    interval: 30s
    rules:
      - record: job:http_requests:rate5m
        expr: rate(http_requests_total[5m])
      
      - record: job:http_errors:rate5m
        expr: rate(http_requests_total{status=~"5.."}[5m])
      
      - record: job:http_error_rate
        expr: |
          job:http_errors:rate5m / job:http_requests:rate5m
```

### Step 2: Use Recording Rules
```yaml
# prometheus.yml
rule_files:
  - "recording-rules.yml"
  - "alerts.yml"
```

## 📊 Monitoring and Debugging

### Check Prometheus Status
```bash
# Check Prometheus health
curl http://localhost:9090/-/healthy

# Check targets
curl http://localhost:9090/api/v1/targets

# Check configuration
curl http://localhost:9090/api/v1/status/config
```

### Debug Scraping Issues
```bash
# View target status
# Prometheus UI > Status > Targets

# Check scrape errors
# Prometheus UI > Status > Targets > Show more

# Test metric endpoint
curl http://localhost:8080/metrics
```

## 🎯 Common Usage Patterns

### 1. **Application Monitoring**
```yaml
# 1. Instrument application
# 2. Expose /metrics endpoint
# 3. Configure Prometheus to scrape
# 4. Create dashboards in Grafana
# 5. Set up alerts
```

### 2. **Infrastructure Monitoring**
```yaml
# 1. Install Node Exporter
# 2. Configure Prometheus scrape
# 3. Query system metrics
# 4. Create infrastructure dashboard
# 5. Set up infrastructure alerts
```

### 3. **Multi-service Monitoring**
```yaml
# 1. Use service discovery
# 2. Configure multiple scrape jobs
# 3. Aggregate metrics by service
# 4. Create service-level dashboards
```

---

*Prometheus is now integrated into your monitoring workflow! 🎉*