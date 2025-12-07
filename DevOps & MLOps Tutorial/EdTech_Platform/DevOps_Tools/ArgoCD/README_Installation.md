# ArgoCD — Installation

## 🚀 Installation Methods

ArgoCD can be installed on Kubernetes clusters using multiple methods.

## 🐳 Method 1: Kubernetes Manifest (Recommended)

### Basic Installation
```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for pods to be ready
kubectl wait --for=condition=ready pod --all -n argocd --timeout=300s
```

### Access ArgoCD
```bash
# Port forward to access UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Access at https://localhost:8080
# Username: admin
# Password: (from above command)
```

## 🐧 Method 2: Helm Chart

### Install via Helm
```bash
# Add ArgoCD Helm repository
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

# Install ArgoCD
helm install argocd argo/argo-cd \
  --namespace argocd \
  --create-namespace \
  --set server.service.type=LoadBalancer
```

## ☁️ Method 3: Cloud Installation

### AWS EKS
```bash
# Install on EKS
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Expose via LoadBalancer
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

## 📋 Prerequisites

### System Requirements
- **Kubernetes**: 1.19+ cluster
- **kubectl**: Configured and connected
- **Storage**: Persistent volume support
- **Network**: Ingress or LoadBalancer

## ⚙️ Installation Steps

### Step 1: Install ArgoCD
```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Step 2: Wait for Installation
```bash
# Check pod status
kubectl get pods -n argocd

# Wait for all pods to be ready
kubectl wait --for=condition=ready pod --all -n argocd --timeout=300s
```

### Step 3: Access ArgoCD
```bash
# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Port forward
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

## ✅ Verification

### Test ArgoCD Installation
```bash
# Check ArgoCD pods
kubectl get pods -n argocd

# Check ArgoCD services
kubectl get svc -n argocd

# Test API access
curl -k https://localhost:8080/api/version
```

---

*ArgoCD is ready for GitOps deployments! 🎉*