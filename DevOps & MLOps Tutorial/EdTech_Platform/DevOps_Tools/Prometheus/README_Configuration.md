# Prometheus — Configuration

## ⚙️ Configuration Overview

Prometheus configuration is managed through the `prometheus.yml` file. The configuration file uses YAML format and defines global settings, scrape configurations, alerting rules, and storage settings.

## 📁 Configuration Files

### 1. **Prometheus Configuration** (`prometheus.yml`)
```yaml
# Global configuration
global:
  scrape_interval: 15s          # How often to scrape targets
  evaluation_interval: 15s      # How often to evaluate rules
  external_labels:
    cluster: 'production'
    environment: 'prod'
    region: 'us-west-2'

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093
    - kubernetes_sd_configs:
        - role: pod
          namespaces:
            names:
              - monitoring

# Rule files
rule_files:
  - "alerts.yml"
  - "recording-rules.yml"

# Scrape configurations
scrape_configs:
  # Scrape Prometheus itself
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
        labels:
          instance: 'prometheus-server'

  # Scrape Node Exporter
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
        labels:
          instance: 'server-1'
          environment: 'production'

  # Scrape Kubernetes pods
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
        namespaces:
          names:
            - default
            - production
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

  # Scrape Kubernetes services
  - job_name: 'kubernetes-services'
    kubernetes_sd_configs:
      - role: service
    relabel_configs:
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
        action: keep
        regex: true

  # Scrape with authentication
  - job_name: 'authenticated-endpoint'
    basic_auth:
      username: 'prometheus'
      password: 'secret-password'
    static_configs:
      - targets: ['secure-endpoint:9090']

  # Scrape with TLS
  - job_name: 'tls-endpoint'
    scheme: https
    tls_config:
      ca_file: /etc/prometheus/certs/ca.crt
      cert_file: /etc/prometheus/certs/client.crt
      key_file: /etc/prometheus/certs/client.key
      insecure_skip_verify: false
    static_configs:
      - targets: ['secure-endpoint:9090']
```

### 2. **Alert Rules** (`alerts.yml`)
```yaml
groups:
  - name: infrastructure
    interval: 30s
    rules:
      - alert: HighCPUUsage
        expr: 100 - (avg(irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: warning
          team: infrastructure
        annotations:
          summary: "High CPU usage detected on {{ $labels.instance }}"
          description: "CPU usage is {{ $value }}% on {{ $labels.instance }} for more than 5 minutes"
      
      - alert: HighMemoryUsage
        expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 90
        for: 5m
        labels:
          severity: critical
          team: infrastructure
        annotations:
          summary: "High memory usage detected"
          description: "Memory usage is {{ $value }}% on {{ $labels.instance }}"
      
      - alert: DiskSpaceLow
        expr: (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100 < 10
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Disk space is running low"
          description: "Disk space is {{ $value }}% available on {{ $labels.instance }}"

  - name: application
    interval: 30s
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.05
        for: 5m
        labels:
          severity: warning
          team: application
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value | humanizePercentage }} for service {{ $labels.service }}"
      
      - alert: ServiceDown
        expr: up{job="my-service"} == 0
        for: 1m
        labels:
          severity: critical
          team: application
        annotations:
          summary: "Service {{ $labels.job }} is down"
          description: "Service {{ $labels.job }} has been down for more than 1 minute"
```

### 3. **Recording Rules** (`recording-rules.yml`)
```yaml
groups:
  - name: recording_rules
    interval: 30s
    rules:
      # Pre-compute request rate
      - record: job:http_requests:rate5m
        expr: rate(http_requests_total[5m])
      
      # Pre-compute error rate
      - record: job:http_errors:rate5m
        expr: rate(http_requests_total{status=~"5.."}[5m])
      
      # Calculate error percentage
      - record: job:http_error_rate
        expr: |
          job:http_errors:rate5m / job:http_requests:rate5m
      
      # Pre-compute CPU usage
      - record: instance:node_cpu_usage
        expr: 100 - (avg(irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
      
      # Pre-compute memory usage
      - record: instance:node_memory_usage
        expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100
```

## 🔧 Basic Configuration

### 1. **Scrape Configuration**
```yaml
scrape_configs:
  - job_name: 'my-app'
    scrape_interval: 30s          # Override global scrape_interval
    scrape_timeout: 10s           # Timeout for scrape requests
    metrics_path: '/metrics'      # Path to metrics endpoint
    scheme: http                  # http or https
    static_configs:
      - targets: ['app:8080']
        labels:
          environment: 'production'
          service: 'my-app'
```

### 2. **Service Discovery**
```yaml
# Kubernetes service discovery
scrape_configs:
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
        namespaces:
          names:
            - default
            - production
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_name]
        target_label: pod
      - source_labels: [__meta_kubernetes_namespace]
        target_label: namespace
```

### 3. **Relabeling**
```yaml
scrape_configs:
  - job_name: 'my-app'
    static_configs:
      - targets: ['app:8080']
    relabel_configs:
      # Add custom label
      - target_label: environment
        replacement: 'production'
      
      # Drop specific metrics
      - source_labels: [__name__]
        regex: 'go_.*'
        action: drop
      
      # Keep only specific metrics
      - source_labels: [__name__]
        regex: 'http_requests_total'
        action: keep
```

## 🔐 Security Configuration

### 1. **Authentication**
```yaml
scrape_configs:
  - job_name: 'authenticated-endpoint'
    basic_auth:
      username: 'prometheus'
      password_file: '/etc/prometheus/password'
    
    # Or use bearer token
    bearer_token_file: '/etc/prometheus/token'
```

### 2. **TLS Configuration**
```yaml
scrape_configs:
  - job_name: 'tls-endpoint'
    scheme: https
    tls_config:
      ca_file: /etc/prometheus/certs/ca.crt
      cert_file: /etc/prometheus/certs/client.crt
      key_file: /etc/prometheus/certs/client.key
      insecure_skip_verify: false
      server_name: 'prometheus.example.com'
```

### 3. **Web Configuration**
```yaml
# Command line flags or configuration
# --web.config.file=web-config.yml

# web-config.yml
tls_server_config:
  cert_file: /etc/prometheus/certs/server.crt
  key_file: /etc/prometheus/certs/server.key

basic_auth_users:
  admin: $2b$12$...
```

## 🔄 Advanced Configuration

### 1. **Remote Write**
```yaml
# Configure remote write
remote_write:
  - url: 'https://prometheus-remote-write.example.com/api/v1/write'
    basic_auth:
      username: 'prometheus'
      password: 'secret'
    queue_config:
      max_samples_per_send: 1000
      max_shards: 200
      capacity: 2500
```

### 2. **Remote Read**
```yaml
# Configure remote read
remote_read:
  - url: 'https://prometheus-remote-read.example.com/api/v1/read'
    basic_auth:
      username: 'prometheus'
      password: 'secret'
```

### 3. **Storage Configuration**
```yaml
# Storage configuration (command line flags)
# --storage.tsdb.path=/var/lib/prometheus
# --storage.tsdb.retention.time=30d
# --storage.tsdb.retention.size=50GB
```

## 🐛 Common Configuration Issues

### Issue 1: Configuration Syntax Error
```bash
# Error: YAML syntax error
# Solution: Validate configuration
promtool check config prometheus.yml

# Expected output:
# Checking prometheus.yml
# SUCCESS: prometheus.yml is valid prometheus config file syntax
```

### Issue 2: Target Not Scraping
```bash
# Problem: Target not being scraped
# Solution: Check target status
curl http://localhost:9090/api/v1/targets

# Check relabeling
promtool test rules alerts.yml
```

### Issue 3: Alert Not Firing
```bash
# Problem: Alert not firing
# Solution: Check alert rules
promtool check rules alerts.yml

# Test alert expression
curl 'http://localhost:9090/api/v1/query?query=up'
```

## ✅ Configuration Validation

### Validate Configuration
```bash
# Check configuration syntax
promtool check config prometheus.yml

# Check alert rules
promtool check rules alerts.yml

# Test rules
promtool test rules alerts.yml

# Check configuration via API
curl http://localhost:9090/api/v1/status/config
```

### Configuration Checklist
- [ ] Global settings configured
- [ ] Scrape targets configured
- [ ] Alertmanager configured
- [ ] Alert rules defined
- [ ] Recording rules defined (optional)
- [ ] Authentication configured (if needed)
- [ ] TLS configured (if needed)
- [ ] Storage retention configured
- [ ] External labels set

---

*Prometheus configuration is now optimized! 🎯*