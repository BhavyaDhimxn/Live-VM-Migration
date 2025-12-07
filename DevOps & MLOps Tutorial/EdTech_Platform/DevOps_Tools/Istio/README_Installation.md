# Istio — Installation

## 🚀 Installation Methods

Istio can be installed on Kubernetes clusters using multiple methods.

## 🐳 Method 1: Istioctl (Recommended)

### Download Istio
```bash
# Download Istio
curl -L https://istio.io/downloadIstio | sh -
cd istio-*

# Add to PATH
export PATH=$PWD/bin:$PATH
```

### Install Istio
```bash
# Install Istio with default profile
istioctl install --set values.defaultRevision=default

# Or install with demo profile
istioctl install --set profile=demo

# Verify installation
istioctl verify-install
```

## 🐧 Method 2: Helm Chart

### Install via Helm
```bash
# Add Istio Helm repository
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update

# Install Istio base
helm install istio-base istio/base -n istio-system --create-namespace

# Install Istiod
helm install istiod istio/istiod -n istio-system --wait

# Install Istio ingress gateway
helm install istio-ingress istio/gateway -n istio-system
```

## ☁️ Method 3: Operator

### Install via Operator
```bash
# Install Istio operator
istioctl operator init

# Create IstioControlPlane
kubectl apply -f - <<EOF
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  name: control-plane
  namespace: istio-system
spec:
  profile: default
EOF
```

## 📋 Prerequisites

### System Requirements
- **Kubernetes**: 1.19+ cluster
- **kubectl**: Configured and connected
- **Storage**: Persistent volume support
- **Network**: CNI plugin support

## ⚙️ Installation Steps

### Step 1: Download Istio
```bash
# Download Istio
curl -L https://istio.io/downloadIstio | sh -
cd istio-*
export PATH=$PWD/bin:$PATH
```

### Step 2: Install Istio
```bash
# Install with default profile
istioctl install --set values.defaultRevision=default

# Or install with demo profile (includes addons)
istioctl install --set profile=demo
```

### Step 3: Enable Sidecar Injection
```bash
# Enable automatic sidecar injection for namespace
kubectl label namespace default istio-injection=enabled

# Verify
kubectl get namespace -L istio-injection
```

### Step 4: Install Addons (Optional)
```bash
# Install Prometheus
kubectl apply -f samples/addons/prometheus.yaml

# Install Grafana
kubectl apply -f samples/addons/grafana.yaml

# Install Kiali
kubectl apply -f samples/addons/kiali.yaml

# Install Jaeger
kubectl apply -f samples/addons/jaeger.yaml
```

## ✅ Verification

### Test Istio Installation
```bash
# Check Istio pods
kubectl get pods -n istio-system

# Verify installation
istioctl verify-install

# Check Istio version
istioctl version

# Check proxy status
istioctl proxy-status
```

---

*Istio is ready for service mesh! 🎉*