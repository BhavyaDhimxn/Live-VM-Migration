# Prometheus — Best Practices

## 🎯 Production Best Practices

### 1. **Metric Naming Convention**
```promql
# Use consistent naming
# Format: {namespace}_{subsystem}_{name}_{unit}
# Examples:
# - http_requests_total (counter)
# - http_request_duration_seconds (histogram)
# - cpu_usage_percent (gauge)
# - memory_bytes (gauge)

# Good metric names:
http_requests_total
http_request_duration_seconds
cpu_usage_percent

# Bad metric names:
requests
duration
cpu
```

### 2. **Label Best Practices**
```promql
# Use labels effectively
# Good labels:
http_requests_total{method="GET", endpoint="/api/users", status="200"}
cpu_usage{instance="server-1", environment="production"}

# Avoid high-cardinality labels:
http_requests_total{user_id="12345"}  # Too many unique values
http_requests_total{request_id="uuid"}  # Too many unique values
```

## 🔐 Security Best Practices

### 1. **Access Control**
```yaml
# Use reverse proxy for authentication
# nginx configuration
server {
    listen 80;
    server_name prometheus.example.com;
    
    location / {
        auth_basic "Prometheus";
        auth_basic_user_file /etc/nginx/.htpasswd;
        proxy_pass http://localhost:9090;
    }
}
```

### 2. **TLS Configuration**
```yaml
# Enable TLS in Prometheus
# Use reverse proxy with TLS
# Or use Prometheus Operator with TLS
```

## 📊 Configuration Best Practices

### 1. **Scrape Configuration**
```yaml
# Optimize scrape intervals
scrape_configs:
  - job_name: 'critical-app'
    scrape_interval: 5s  # More frequent for critical apps
  
  - job_name: 'background-job'
    scrape_interval: 60s  # Less frequent for background jobs
```

### 2. **Retention Configuration**
```yaml
# Configure retention
# Command line:
prometheus --storage.tsdb.retention.time=30d

# Or in docker-compose:
command:
  - '--storage.tsdb.retention.time=30d'
```

## 🔔 Alerting Best Practices

### 1. **Alert Rule Design**
```yaml
# Design effective alerts
groups:
  - name: application
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.05
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value | humanizePercentage }}"
```

### 2. **Alert Grouping**
```yaml
# Group related alerts
route:
  group_by: ['alertname', 'cluster', 'service']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
```

## ⚡ Performance Optimization

### 1. **Query Optimization**
```promql
# Use rate() for counters
rate(http_requests_total[5m])

# Use appropriate time ranges
rate(http_requests_total[5m])  # Good
rate(http_requests_total[1h])   # Too long

# Limit series cardinality
http_requests_total{method="GET"}  # Filtered
http_requests_total  # All series
```

### 2. **Recording Rules**
```yaml
# Pre-compute expensive queries
groups:
  - name: recording_rules
    rules:
      - record: job:http_requests:rate5m
        expr: rate(http_requests_total[5m])
```

## 🔧 Reproducibility Best Practices

### 1. **Configuration Versioning**
```bash
# Version control Prometheus configuration
git init
git add prometheus.yml alerts.yml
git commit -m "Add Prometheus configuration"
```

### 2. **Dashboard as Code**
```bash
# Export Grafana dashboards
# Store in version control
# Deploy via CI/CD
```

## 🧪 Testing Best Practices

### 1. **Alert Testing**
```bash
# Test alert rules
# Prometheus UI > Alerts > Test rule

# Use alert testing tool
amtool check-config alertmanager.yml
```

### 2. **Query Testing**
```bash
# Test queries in Prometheus UI
# Prometheus UI > Graph > Enter query

# Validate query syntax
promtool check rules alerts.yml
```

## 📚 Documentation Best Practices

### 1. **Metric Documentation**
```yaml
# Document metrics
# Use HELP and TYPE in metrics
# HELP http_requests_total Total number of HTTP requests
# TYPE http_requests_total counter
```

### 2. **Alert Documentation**
```yaml
# Document alerts
- alert: HighCPUUsage
  expr: cpu_usage > 80
  annotations:
    summary: "High CPU usage"
    description: "CPU usage is {{ $value }}% on {{ $labels.instance }}"
    runbook_url: "https://wiki.example.com/alerts/high-cpu"
```

## 🚨 Common Pitfalls to Avoid

### 1. **Don't Use High-cardinality Labels**
```promql
# Bad: High-cardinality label
http_requests_total{user_id="12345"}

# Good: Aggregate by service
http_requests_total{service="api"}
```

### 2. **Don't Scrape Too Frequently**
```yaml
# Bad: Too frequent scraping
scrape_interval: 1s

# Good: Appropriate interval
scrape_interval: 15s
```

### 3. **Don't Ignore Retention**
```bash
# Bad: No retention limit
# Data grows indefinitely

# Good: Set retention
prometheus --storage.tsdb.retention.time=30d
```

---

*Follow these best practices to build effective monitoring with Prometheus! 🎯*