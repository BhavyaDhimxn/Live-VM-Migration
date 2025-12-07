# Grafana — Configuration

## ⚙️ Configuration Overview

Grafana configuration is managed through the `grafana.ini` file and environment variables.

## 📁 Configuration Files

### 1. **Grafana Configuration** (`/etc/grafana/grafana.ini`)
```ini
[server]
http_port = 3000
domain = localhost
root_url = http://localhost:3000

[database]
type = sqlite3
path = grafana.db

[security]
admin_user = admin
admin_password = admin
secret_key = your-secret-key

[datasources]
# Data source configurations

[alerting]
enabled = true
```

### 2. **Environment Variables**
```bash
# Set Grafana configuration
export GF_SERVER_HTTP_PORT=3000
export GF_SECURITY_ADMIN_PASSWORD=admin
export GF_DATABASE_TYPE=postgres
```

## 🔧 Basic Configuration

### 1. **Server Configuration**
```ini
[server]
http_port = 3000
domain = grafana.example.com
root_url = https://grafana.example.com
```

### 2. **Database Configuration**
```ini
[database]
type = postgres
host = localhost:5432
name = grafana
user = grafana
password = password
```

### 3. **Security Configuration**
```ini
[security]
admin_user = admin
admin_password = secure-password
secret_key = your-secret-key
disable_gravatar = false
```

## 🔐 Security Configuration

### 1. **Authentication**
```ini
[auth.anonymous]
enabled = false

[auth.github]
enabled = true
client_id = your-client-id
client_secret = your-client-secret
```

### 2. **HTTPS Configuration**
```ini
[server]
protocol = https
cert_file = /path/to/cert.pem
cert_key = /path/to/key.pem
```

---

*Grafana configuration optimized! 🎯*