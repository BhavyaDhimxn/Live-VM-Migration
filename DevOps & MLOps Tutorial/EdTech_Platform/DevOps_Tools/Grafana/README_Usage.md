# Grafana — Usage

## 🚀 Getting Started with Grafana

This guide covers practical usage examples for Grafana in real-world monitoring scenarios.

## 📊 Example 1: Create Monitoring Dashboard

### Scenario: Monitor Server Infrastructure

Create a comprehensive dashboard to monitor server CPU, memory, disk, and network metrics.

### Step 1: Add Prometheus Data Source
```bash
# Via Web UI:
# 1. Configuration > Data Sources > Add data source
# 2. Select Prometheus
# 3. Configure:
#    - URL: http://prometheus:9090
#    - Access: Server (default)
# 4. Click "Save & Test"
```

### Step 2: Create CPU Usage Panel
```json
{
  "dashboard": {
    "title": "Server Infrastructure Monitoring",
    "panels": [
      {
        "id": 1,
        "title": "CPU Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "100 - (avg(irate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)",
            "legendFormat": "CPU Usage %"
          }
        ],
        "gridPos": {
          "h": 8,
          "w": 12,
          "x": 0,
          "y": 0
        }
      }
    ]
  }
}
```

### Step 3: Add Memory Panel
```json
{
  "id": 2,
  "title": "Memory Usage",
  "type": "graph",
  "targets": [
    {
      "expr": "(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100",
      "legendFormat": "Memory Usage %"
    }
  ]
}
```

### Step 4: Add Disk Panel
```json
{
  "id": 3,
  "title": "Disk Usage",
  "type": "graph",
  "targets": [
    {
      "expr": "100 - ((node_filesystem_avail_bytes{mountpoint=\"/\"} * 100) / node_filesystem_size_bytes{mountpoint=\"/\"})",
      "legendFormat": "Disk Usage %"
    }
  ]
}
```

## 🔧 Example 2: Application Monitoring Dashboard

### Scenario: Monitor Application Performance

Create a dashboard to monitor application metrics like request rate, error rate, and latency.

### Step 1: Configure Application Metrics
```json
{
  "dashboard": {
    "title": "Application Performance",
    "panels": [
      {
        "id": 1,
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "legendFormat": "{{method}} {{endpoint}}"
          }
        ]
      },
      {
        "id": 2,
        "title": "Error Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total{status=~\"5..\"}[5m])",
            "legendFormat": "5xx Errors"
          }
        ],
        "alert": {
          "conditions": [
            {
              "evaluator": {
                "params": [0.1],
                "type": "gt"
              },
              "operator": {
                "type": "and"
              },
              "query": {
                "params": ["A", "5m", "now"]
              },
              "type": "query"
            }
          ]
        }
      },
      {
        "id": 3,
        "title": "Response Time",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "95th percentile"
          }
        ]
      }
    ]
  }
}
```

## 🚀 Example 3: Alert Configuration

### Scenario: Set Up Alerts for Critical Metrics

Configure alerts to notify when metrics exceed thresholds.

### Step 1: Create Alert Rule
```json
{
  "alert": {
    "name": "High CPU Usage",
    "message": "CPU usage is above 80%",
    "conditions": [
      {
        "evaluator": {
          "params": [80],
          "type": "gt"
        },
        "operator": {
          "type": "and"
        },
        "query": {
          "params": [
            "A",
            "5m",
            "now"
          ]
        },
        "type": "query"
      }
    ],
    "executionErrorState": "alerting",
    "for": "5m",
    "frequency": "10s",
    "handler": 1,
    "noDataState": "no_data",
    "notifications": []
  }
}
```

### Step 2: Configure Notification Channels
```bash
# Via Web UI:
# 1. Alerting > Notification channels > New channel
# 2. Select channel type (Email, Slack, PagerDuty, etc.)
# 3. Configure channel settings
# 4. Test notification
```

### Step 3: Link Alert to Dashboard
```json
{
  "panel": {
    "alert": {
      "name": "High CPU Alert",
      "conditions": [
        {
          "evaluator": {
            "params": [80],
            "type": "gt"
          }
        }
      ],
      "notifications": [
        {
          "uid": "email-channel"
        }
      ]
    }
  }
}
```

## 🎯 Example 4: Multi-source Dashboard

### Scenario: Combine Data from Multiple Sources

Create a dashboard that combines data from Prometheus, InfluxDB, and CloudWatch.

### Step 1: Add Multiple Data Sources
```bash
# Add Prometheus data source
# Configuration > Data Sources > Add > Prometheus

# Add InfluxDB data source
# Configuration > Data Sources > Add > InfluxDB

# Add CloudWatch data source
# Configuration > Data Sources > Add > CloudWatch
```

### Step 2: Create Mixed Data Source Dashboard
```json
{
  "dashboard": {
    "title": "Multi-source Monitoring",
    "panels": [
      {
        "title": "Prometheus Metrics",
        "datasource": "Prometheus",
        "targets": [
          {
            "expr": "up"
          }
        ]
      },
      {
        "title": "InfluxDB Metrics",
        "datasource": "InfluxDB",
        "targets": [
          {
            "query": "SELECT mean(\"value\") FROM \"cpu\" WHERE $timeRange"
          }
        ]
      },
      {
        "title": "CloudWatch Metrics",
        "datasource": "CloudWatch",
        "targets": [
          {
            "namespace": "AWS/EC2",
            "metricName": "CPUUtilization",
            "statistics": ["Average"]
          }
        ]
      }
    ]
  }
}
```

## 🔄 Example 5: Dashboard Variables

### Scenario: Create Dynamic Dashboards

Use variables to make dashboards dynamic and reusable.

### Step 1: Define Variables
```json
{
  "dashboard": {
    "templating": {
      "list": [
        {
          "name": "environment",
          "type": "query",
          "query": "label_values(up, environment)",
          "current": {
            "value": "production"
          }
        },
        {
          "name": "server",
          "type": "query",
          "query": "label_values(up{environment=\"$environment\"}, instance)",
          "multi": true,
          "includeAll": true
        }
      ]
    }
  }
}
```

### Step 2: Use Variables in Queries
```json
{
  "targets": [
    {
      "expr": "cpu_usage{environment=\"$environment\", instance=~\"$server\"}",
      "legendFormat": "{{instance}}"
    }
  ]
}
```

## 📊 Monitoring and Debugging

### Check Dashboard Performance
```bash
# View dashboard performance
# Dashboard Settings > General > View JSON
# Check query execution time in panel inspector
```

### Debug Data Source Issues
```bash
# Test data source connection
# Configuration > Data Sources > Test connection

# Check query results
# Panel > Inspect > Query
```

## 🎯 Common Usage Patterns

### 1. **Infrastructure Monitoring**
```bash
# 1. Add Prometheus data source
# 2. Create dashboard with system metrics
# 3. Set up alerts for critical metrics
# 4. Share dashboard with team
```

### 2. **Application Monitoring**
```bash
# 1. Instrument application with metrics
# 2. Export metrics to Prometheus
# 3. Create application dashboard
# 4. Configure application-specific alerts
```

### 3. **Business Metrics**
```bash
# 1. Connect to business data source
# 2. Create business KPI dashboard
# 3. Set up business alerts
# 4. Schedule reports
```

---

*Grafana is now integrated into your monitoring workflow! 🎉*