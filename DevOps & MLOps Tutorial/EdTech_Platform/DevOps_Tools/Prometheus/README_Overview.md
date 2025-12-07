# Prometheus — Overview

## 🎯 What is Prometheus?

**Prometheus** is an open-source monitoring and alerting toolkit designed for reliability and scalability. It collects metrics from configured targets at given intervals, evaluates rule expressions, displays results, and can trigger alerts if some condition is observed to be true.

## 🧩 Role in DevOps Lifecycle

Prometheus plays a crucial role in the **Monitoring** and **Observability** stages of the DevOps lifecycle:

- **📊 Metrics Collection**: Collect metrics from applications and infrastructure
- **🔍 Service Discovery**: Automatically discover monitoring targets
- **📈 Time-series Storage**: Store metrics as time-series data
- **🔔 Alerting**: Trigger alerts based on metric conditions
- **📉 Query Language**: Powerful PromQL for querying metrics
- **🔄 Pull Model**: Pull metrics from targets at regular intervals

## 🚀 Key Components

### 1. **Prometheus Server**
```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

### 2. **Exporters**
```bash
# Node Exporter for system metrics
docker run -d -p 9100:9100 prom/node-exporter

# Application exporters
# - MySQL Exporter
# - PostgreSQL Exporter
# - Redis Exporter
```

### 3. **Alertmanager**
```yaml
# alertmanager.yml
route:
  group_by: ['alertname', 'cluster']
  receiver: 'default-receiver'
  routes:
    - match:
        severity: critical
      receiver: 'critical-receiver'

receivers:
  - name: 'default-receiver'
    email_configs:
      - to: 'team@example.com'
```

### 4. **PromQL (Query Language)**
```promql
# Query examples
up
rate(http_requests_total[5m])
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

## ⚙️ When to Use Prometheus

### ✅ **Perfect For:**
- **Metrics Monitoring**: Time-series metrics collection
- **Service Monitoring**: Monitor microservices and applications
- **Infrastructure Monitoring**: Monitor servers, containers, networks
- **Alerting**: Set up alerting based on metrics
- **Multi-dimensional Data**: Labels for flexible querying
- **Pull-based Monitoring**: Active metric collection

### ❌ **Not Ideal For:**
- **Log Storage**: Not designed for log storage
- **Event Storage**: Not for event streaming
- **High-cardinality Data**: Limited for very high cardinality
- **Long-term Storage**: Better with external storage for long-term

## 💡 Key Differentiators

| Feature | Prometheus | Other Monitoring |
|---------|------------|-----------------|
| **Pull Model** | ✅ Active | ⚠️ Push-based |
| **PromQL** | ✅ Powerful | ⚠️ Limited |
| **Multi-dimensional** | ✅ Labels | ⚠️ Fixed |
| **Service Discovery** | ✅ Built-in | ⚠️ Manual |
| **Open Source** | ✅ Free | ❌ Commercial |

## 🔗 Integration Ecosystem

### Exporters
- **Node Exporter**: System metrics
- **cAdvisor**: Container metrics
- **JMX Exporter**: Java applications
- **Blackbox Exporter**: Endpoint monitoring

### Alerting
- **Alertmanager**: Alert routing and notification
- **Grafana**: Visualization and alerting
- **PagerDuty**: Incident management

---

*Prometheus provides powerful monitoring and alerting capabilities! 🎯*