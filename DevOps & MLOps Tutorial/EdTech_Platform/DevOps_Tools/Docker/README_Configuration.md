# Docker — Configuration

## ⚙️ Configuration Overview

Docker configuration is managed through the `daemon.json` file and environment variables to customize Docker behavior.

## 📁 Configuration Files

### 1. **Docker Daemon Configuration** (`/etc/docker/daemon.json`)
```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "storage-driver": "overlay2",
  "data-root": "/var/lib/docker",
  "hosts": ["unix:///var/run/docker.sock", "tcp://0.0.0.0:2376"],
  "tls": true,
  "tlsverify": true,
  "tlscacert": "/etc/docker/ca.pem",
  "tlscert": "/etc/docker/server.pem",
  "tlskey": "/etc/docker/server-key.pem"
}
```

### 2. **Docker Client Configuration** (`~/.docker/config.json`)
```json
{
  "auths": {
    "registry.example.com": {
      "username": "user",
      "password": "password"
    }
  },
  "HttpHeaders": {
    "User-Agent": "Docker-Client/24.0.0"
  }
}
```

## 🔧 Basic Configuration

### 1. **Logging Configuration**
```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

### 2. **Storage Configuration**
```json
{
  "storage-driver": "overlay2",
  "data-root": "/var/lib/docker"
}
```

### 3. **Network Configuration**
```json
{
  "bip": "172.17.0.1/16",
  "default-address-pools": [
    {
      "base": "172.20.0.0/16",
      "size": 24
    }
  ]
}
```

## 🔐 Security Configuration

### 1. **TLS Configuration**
```json
{
  "tls": true,
  "tlsverify": true,
  "tlscacert": "/etc/docker/ca.pem",
  "tlscert": "/etc/docker/server.pem",
  "tlskey": "/etc/docker/server-key.pem"
}
```

### 2. **User Namespace**
```json
{
  "userns-remap": "default"
}
```

## 🔄 Advanced Configuration

### 1. **Registry Configuration**
```json
{
  "insecure-registries": ["registry.example.com:5000"],
  "registry-mirrors": ["https://mirror.example.com"]
}
```

### 2. **Resource Limits**
```json
{
  "default-ulimits": {
    "nofile": {
      "Name": "nofile",
      "Hard": 64000,
      "Soft": 64000
    }
  }
}
```

---

*Your Docker configuration is now optimized! 🎯*