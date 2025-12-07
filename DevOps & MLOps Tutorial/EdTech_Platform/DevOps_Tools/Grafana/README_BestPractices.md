# Grafana — Best Practices

## 🎯 Production Best Practices

### 1. **Dashboard Organization**
```
grafana/
├── dashboards/
│   ├── infrastructure/
│   │   ├── servers.json
│   │   ├── networks.json
│   │   └── databases.json
│   ├── applications/
│   │   ├── web-app.json
│   │   └── api-service.json
│   └── business/
│       ├── kpis.json
│       └── revenue.json
├── datasources/
│   ├── prometheus.yaml
│   └── influxdb.yaml
└── alerts/
    ├── infrastructure-alerts.json
    └── application-alerts.json
```

### 2. **Dashboard Naming Convention**
```bash
# Consistent dashboard naming
# Format: {category}_{purpose}_{environment}
# Examples:
# - infrastructure_servers_production
# - application_webapp_staging
# - business_kpis_production
```

## 🔐 Security Best Practices

### 1. **Authentication Configuration**
```ini
# Use strong authentication
[auth.github]
enabled = true
client_id = your-client-id
client_secret = your-client-secret
scopes = user:email,read:org
auth_url = https://github.com/login/oauth/authorize
token_url = https://github.com/login/oauth/access_token
api_url = https://api.github.com/user
team_ids = 123,456

# Or use LDAP
[auth.ldap]
enabled = true
config_file = /etc/grafana/ldap.toml
```

### 2. **Access Control**
```ini
# Configure user permissions
[auth]
disable_login_form = false
disable_signout_menu = false

# Use role-based access
[users]
allow_sign_up = false
allow_org_create = false
```

### 3. **Data Source Security**
```ini
# Secure data source access
[datasources]
# Use credentials from environment variables
# Never hardcode credentials in configuration
```

## 📊 Dashboard Best Practices

### 1. **Panel Organization**
```json
{
  "dashboard": {
    "panels": [
      {
        "title": "Overview Metrics",
        "gridPos": {"h": 4, "w": 6, "x": 0, "y": 0}
      },
      {
        "title": "Detailed Metrics",
        "gridPos": {"h": 8, "w": 12, "x": 0, "y": 4}
      }
    ],
    "refresh": "30s",
    "time": {
      "from": "now-6h",
      "to": "now"
    }
  }
}
```

### 2. **Query Optimization**
```json
{
  "targets": [
    {
      "expr": "rate(http_requests_total[5m])",
      "legendFormat": "{{method}}",
      "interval": "30s",
      "maxDataPoints": 1000
    }
  ]
}
```

### 3. **Dashboard Variables**
```json
{
  "templating": {
    "list": [
      {
        "name": "environment",
        "type": "query",
        "query": "label_values(up, environment)",
        "refresh": 1,
        "regex": "",
        "sort": 0
      }
    ]
  }
}
```

## 🔔 Alerting Best Practices

### 1. **Alert Configuration**
```json
{
  "alert": {
    "name": "High Error Rate",
    "message": "Error rate exceeded threshold",
    "conditions": [
      {
        "evaluator": {
          "params": [0.05],
          "type": "gt"
        },
        "operator": {
          "type": "and"
        },
        "query": {
          "params": ["A", "5m", "now"]
        },
        "reducer": {
          "params": [],
          "type": "last"
        },
        "type": "query"
      }
    ],
    "executionErrorState": "alerting",
    "for": "5m",
    "frequency": "10s",
    "handler": 1,
    "noDataState": "no_data",
    "notifications": ["email", "slack"]
  }
}
```

### 2. **Alert Grouping**
```json
{
  "alert": {
    "notifications": [
      {
        "uid": "slack-channel",
        "groupBy": ["alertname", "cluster", "service"],
        "groupWait": "10s",
        "groupInterval": "10s",
        "repeatInterval": "12h"
      }
    ]
  }
}
```

## 🔄 CI/CD Integration

### 1. **Dashboard as Code**
```yaml
# .github/workflows/grafana-dashboards.yml
name: Deploy Grafana Dashboards

on:
  push:
    branches: [main]
    paths:
      - 'dashboards/**'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    
    - name: Deploy dashboards
      run: |
        # Use Grafana API to deploy dashboards
        for dashboard in dashboards/*.json; do
          curl -X POST \
            -H "Content-Type: application/json" \
            -H "Authorization: Bearer $GRAFANA_API_KEY" \
            -d @$dashboard \
            http://grafana:3000/api/dashboards/db
        done
```

## 📈 Performance Optimization

### 1. **Query Optimization**
```json
{
  "targets": [
    {
      "expr": "rate(metric[5m])",  // Use rate for counters
      "interval": "30s",  // Appropriate interval
      "maxDataPoints": 1000  // Limit data points
    }
  ]
}
```

### 2. **Dashboard Refresh Rates**
```json
{
  "dashboard": {
    "refresh": "30s",  // Appropriate refresh rate
    "time": {
      "from": "now-6h",  // Reasonable time range
      "to": "now"
    }
  }
}
```

### 3. **Data Source Optimization**
```ini
# Optimize data source queries
[datasources]
# Use query caching where available
# Limit query time ranges
# Use appropriate aggregation
```

## 🔧 Reproducibility Best Practices

### 1. **Dashboard Versioning**
```bash
# Version control dashboards
git init
git add dashboards/
git commit -m "Add monitoring dashboards"

# Export dashboards
# Dashboard Settings > JSON Model > Copy JSON
```

### 2. **Dashboard Templates**
```json
{
  "dashboard": {
    "templating": {
      "list": [
        {
          "name": "environment",
          "type": "custom",
          "options": [
            {"text": "Production", "value": "prod"},
            {"text": "Staging", "value": "staging"}
          ]
        }
      ]
    }
  }
}
```

## 🧪 Testing Best Practices

### 1. **Dashboard Testing**
```bash
# Test dashboard queries
# Panel > Inspect > Query
# Verify query results

# Test alert rules
# Alerting > Alert rules > Test rule
```

### 2. **Alert Testing**
```bash
# Test alert notifications
# Alerting > Notification channels > Test
# Verify notifications are received
```

## 📚 Documentation Best Practices

### 1. **Dashboard Documentation**
```json
{
  "dashboard": {
    "title": "Server Monitoring",
    "description": "Monitor server CPU, memory, disk, and network metrics",
    "tags": ["infrastructure", "servers", "monitoring"],
    "annotations": {
      "list": [
        {
          "name": "Deployments",
          "datasource": "Prometheus",
          "enable": true
        }
      ]
    }
  }
}
```

### 2. **Panel Documentation**
```json
{
  "panel": {
    "title": "CPU Usage",
    "description": "CPU usage percentage across all servers",
    "targets": [
      {
        "expr": "100 - (avg(irate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)",
        "legendFormat": "CPU Usage %"
      }
    ]
  }
}
```

## 🚨 Common Pitfalls to Avoid

### 1. **Don't Overload Dashboards**
```json
// Bad: Too many panels
{
  "dashboard": {
    "panels": [/* 50+ panels */]
  }
}

// Good: Organized dashboards
{
  "dashboard": {
    "panels": [/* 10-15 focused panels */]
  }
}
```

### 2. **Don't Use Inefficient Queries**
```json
// Bad: Inefficient query
{
  "expr": "metric[1d]"  // Too long time range
}

// Good: Optimized query
{
  "expr": "rate(metric[5m])",  // Use rate for counters
  "interval": "30s"  // Appropriate interval
}
```

### 3. **Don't Hardcode Values**
```json
// Bad: Hardcoded values
{
  "expr": "metric{environment=\"production\"}"
}

// Good: Use variables
{
  "expr": "metric{environment=\"$environment\"}"
}
```

---

*Follow these best practices to build effective monitoring dashboards with Grafana! 🎯*