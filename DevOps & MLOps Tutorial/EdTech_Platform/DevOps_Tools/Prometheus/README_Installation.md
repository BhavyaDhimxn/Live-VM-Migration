# Prometheus — Installation

## 🚀 Installation Methods

Prometheus can be installed using multiple methods depending on your environment and requirements.

## 🐳 Method 1: Docker (Recommended for Development)

### Basic Docker Installation
```bash
# Run Prometheus container
docker run -d \
  --name prometheus \
  -p 9090:9090 \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus:latest

# Access Prometheus at http://localhost:9090
```

### Docker Compose
```yaml
# docker-compose.yml
version: '3.8'
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=30d'
      - '--web.console.libraries=/usr/share/prometheus/console_libraries'
      - '--web.console.templates=/usr/share/prometheus/consoles'
    restart: unless-stopped

volumes:
  prometheus-data:
    driver: local
```

### Run with Docker Compose
```bash
# Start Prometheus
docker-compose up -d

# Check logs
docker-compose logs -f prometheus

# Stop Prometheus
docker-compose down
```

## 🐧 Method 2: Linux Binary Installation

### Download and Install
```bash
# Download latest Prometheus
cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz

# Extract
tar xvfz prometheus-2.45.0.linux-amd64.tar.gz
cd prometheus-2.45.0.linux-amd64

# Create Prometheus user
sudo useradd --no-create-home --shell /bin/false prometheus

# Create directories
sudo mkdir -p /etc/prometheus
sudo mkdir -p /var/lib/prometheus

# Copy binaries
sudo cp prometheus /usr/local/bin/
sudo cp promtool /usr/local/bin/

# Copy configuration
sudo cp -r consoles /etc/prometheus
sudo cp -r console_libraries /etc/prometheus
sudo cp prometheus.yml /etc/prometheus/

# Set ownership
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown -R prometheus:prometheus /var/lib/prometheus
```

### Create Systemd Service
```bash
# Create systemd service file
sudo tee /etc/systemd/system/prometheus.service > /dev/null <<EOF
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus/ \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.enable-lifecycle

[Install]
WantedBy=multi-user.target
EOF

# Reload systemd
sudo systemctl daemon-reload

# Enable and start Prometheus
sudo systemctl enable prometheus
sudo systemctl start prometheus

# Check status
sudo systemctl status prometheus
```

## 🍎 Method 3: macOS Installation

### Homebrew
```bash
# Install Prometheus
brew install prometheus

# Start Prometheus
brew services start prometheus

# Check status
brew services list | grep prometheus
```

### Manual Installation
```bash
# Download
wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.darwin-amd64.tar.gz

# Extract
tar xvfz prometheus-2.45.0.darwin-amd64.tar.gz
cd prometheus-2.45.0.darwin-amd64

# Run Prometheus
./prometheus --config.file=prometheus.yml
```

## ☁️ Method 4: Kubernetes Installation

### Using Helm
```bash
# Add Prometheus Helm repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install Prometheus
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace

# Check installation
kubectl get pods -n monitoring
```

### Using Manifests
```bash
# Create namespace
kubectl create namespace monitoring

# Apply Prometheus manifests
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/bundle.yaml

# Create Prometheus instance
kubectl apply -f - <<EOF
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: prometheus
  namespace: monitoring
spec:
  serviceAccountName: prometheus
  replicas: 1
  retention: 30d
EOF
```

## 📦 Method 5: Package Managers

### Ubuntu/Debian
```bash
# Add Prometheus repository
sudo apt-get install -y apt-transport-https ca-certificates curl
curl -fsSL https://packages.grafana.com/gpg.key | sudo gpg --dearmor -o /usr/share/keyrings/prometheus-archive-keyring.gpg

# Add repository (Note: Prometheus doesn't have official apt repo, use binary or Docker)
# For Ubuntu, use binary installation or Docker
```

### CentOS/RHEL
```bash
# Install from EPEL or use binary installation
# Or use Docker/Podman
```

## 📋 Prerequisites

### System Requirements
- **Memory**: 512MB+ RAM (2GB+ recommended)
- **Storage**: 1GB+ free space (more for data retention)
- **Network**: Internet access for scraping targets
- **Port**: 9090 (default, configurable)
- **Operating System**: Linux, macOS, or Windows

### Required Tools
```bash
# Docker (if using Docker method)
docker --version

# kubectl (if using Kubernetes)
kubectl version --client

# wget or curl (for downloading)
wget --version
```

## ⚙️ Installation Steps

### Step 1: Choose Installation Method
```bash
# Choose based on your environment:
# - Docker: For development and testing
# - Binary: For production Linux servers
# - Kubernetes: For containerized environments
# - Package Manager: For easy installation
```

### Step 2: Create Configuration File
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

### Step 3: Install Prometheus
```bash
# Follow method-specific instructions above
# Example for Docker:
docker run -d \
  --name prometheus \
  -p 9090:9090 \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus:latest
```

### Step 4: Verify Installation
```bash
# Check Prometheus health
curl http://localhost:9090/-/healthy

# Check Prometheus readiness
curl http://localhost:9090/-/ready

# Access web UI
# Open http://localhost:9090 in browser
```

## ✅ Verification

### Test Prometheus Installation
```bash
# Check Prometheus version
prometheus --version

# Or via API
curl http://localhost:9090/api/v1/status/buildinfo

# Check targets
curl http://localhost:9090/api/v1/targets

# Check configuration
curl http://localhost:9090/api/v1/status/config

# Query metrics
curl 'http://localhost:9090/api/v1/query?query=up'
```

### Verify Web UI
```bash
# Access Prometheus web UI
# http://localhost:9090

# Test queries in UI:
# - up
# - prometheus_build_info
# - prometheus_config_last_reload_successful
```

## 🔧 Post-Installation Configuration

### Configure Firewall
```bash
# Allow Prometheus port
sudo ufw allow 9090/tcp

# Or for firewalld
sudo firewall-cmd --permanent --add-port=9090/tcp
sudo firewall-cmd --reload
```

### Configure Reverse Proxy (Optional)
```nginx
# nginx configuration
server {
    listen 80;
    server_name prometheus.example.com;
    
    location / {
        proxy_pass http://localhost:9090;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## 🐛 Troubleshooting

### Issue 1: Port Already in Use
```bash
# Problem: Port 9090 already in use
# Solution: Change port
prometheus --web.listen-address=0.0.0.0:9091

# Or stop conflicting service
sudo systemctl stop <service-name>
```

### Issue 2: Permission Denied
```bash
# Problem: Permission denied
# Solution: Check file permissions
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown -R prometheus:prometheus /var/lib/prometheus
```

### Issue 3: Configuration Error
```bash
# Problem: Configuration file error
# Solution: Validate configuration
promtool check config prometheus.yml

# Test configuration
promtool test rules alerts.yml
```

## 📚 Next Steps

After installation:
1. **Configure Targets**: Add scrape targets
2. **Set Up Alerting**: Configure Alertmanager
3. **Install Exporters**: Node exporter, application exporters
4. **Configure Retention**: Set data retention policy
5. **Set Up Dashboards**: Integrate with Grafana

---

*Prometheus is ready for monitoring! 🎉*