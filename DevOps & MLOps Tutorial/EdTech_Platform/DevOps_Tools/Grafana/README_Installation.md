# Grafana — Installation

## 🚀 Installation Methods

Grafana can be installed using multiple methods depending on your environment.

## 🐳 Method 1: Docker (Recommended)

### Basic Docker Installation
```bash
# Run Grafana container
docker run -d \
  --name grafana \
  -p 3000:3000 \
  -v grafana-storage:/var/lib/grafana \
  grafana/grafana:latest

# Access Grafana at http://localhost:3000
# Default credentials: admin/admin
```

### Docker Compose
```yaml
# docker-compose.yml
version: '3.8'
services:
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    volumes:
      - grafana-storage:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin

volumes:
  grafana-storage:
```

## 🍎 Method 2: macOS

### Homebrew
```bash
brew install grafana
brew services start grafana
```

## 🐧 Method 3: Linux

### Ubuntu/Debian
```bash
# Add Grafana repository
sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

# Install Grafana
sudo apt-get update
sudo apt-get install grafana

# Start Grafana
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

### CentOS/RHEL
```bash
# Add Grafana repository
cat > /etc/yum.repos.d/grafana.repo << EOF
[grafana]
name=grafana
baseurl=https://packages.grafana.com/oss/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
EOF

# Install Grafana
sudo yum install grafana

# Start Grafana
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

## ☁️ Method 4: Cloud Installation

### AWS
```bash
# Deploy Grafana on EC2
# Or use AWS Managed Grafana
aws grafana create-workspace --name my-workspace
```

### Google Cloud
```bash
# Deploy on GCE
# Or use Google Cloud Monitoring with Grafana
```

## 📋 Prerequisites

### System Requirements
- **Memory**: 512MB+ RAM (2GB+ recommended)
- **Storage**: 1GB+ free space
- **Network**: Internet access for data sources
- **Port**: 3000 (default)

## ⚙️ Installation Steps

### Step 1: Install Grafana
```bash
# Choose appropriate method
# Docker: docker run grafana/grafana
# Linux: Follow distribution-specific instructions
```

### Step 2: Access Grafana
```bash
# Open browser
# http://localhost:3000

# Default credentials:
# Username: admin
# Password: admin
# (Change on first login)
```

### Step 3: Configure Data Source
```bash
# Via Web UI: Configuration > Data Sources
# Add data source (e.g., Prometheus, InfluxDB)
```

## ✅ Verification

### Test Grafana Installation
```bash
# Check Grafana status
# Docker:
docker ps | grep grafana

# Linux:
sudo systemctl status grafana-server

# Access web UI
curl http://localhost:3000/api/health
```

---

*Grafana is ready for visualization! 🎉*