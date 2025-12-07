# Kubernetes — Installation

## 🚀 Installation Methods

Kubernetes can be installed using multiple methods depending on your environment and requirements.

## 🐳 Method 1: Minikube (Local Development)

### Installation on macOS
```bash
# Install Minikube
brew install minikube

# Start Minikube
minikube start

# Verify installation
kubectl get nodes

# Access Kubernetes dashboard
minikube dashboard
```

### Installation on Linux
```bash
# Download Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

# Install Minikube
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start Minikube
minikube start --driver=docker

# Verify installation
kubectl get nodes
```

### Installation on Windows
```powershell
# Using Chocolatey
choco install minikube

# Or download from GitHub releases
# https://github.com/kubernetes/minikube/releases

# Start Minikube
minikube start
```

### Minikube Configuration
```bash
# Start with specific Kubernetes version
minikube start --kubernetes-version=v1.28.0

# Start with more resources
minikube start --memory=4096 --cpus=2

# Start with specific driver
minikube start --driver=virtualbox

# Stop Minikube
minikube stop

# Delete Minikube cluster
minikube delete
```

## ☁️ Method 2: Cloud Kubernetes Services

### AWS EKS (Elastic Kubernetes Service)
```bash
# Install eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

# Create EKS cluster
eksctl create cluster \
  --name my-cluster \
  --region us-west-2 \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 2 \
  --nodes-max 5

# Update kubeconfig
aws eks update-kubeconfig --name my-cluster --region us-west-2

# Verify
kubectl get nodes
```

### Google GKE (Google Kubernetes Engine)
```bash
# Install gcloud CLI
# https://cloud.google.com/sdk/docs/install

# Authenticate
gcloud auth login

# Set project
gcloud config set project PROJECT_ID

# Create GKE cluster
gcloud container clusters create my-cluster \
  --zone us-central1-a \
  --num-nodes 3 \
  --machine-type n1-standard-1

# Get credentials
gcloud container clusters get-credentials my-cluster --zone us-central1-a

# Verify
kubectl get nodes
```

### Azure AKS (Azure Kubernetes Service)
```bash
# Install Azure CLI
# https://docs.microsoft.com/en-us/cli/azure/install-azure-cli

# Login
az login

# Create resource group
az group create --name myResourceGroup --location eastus

# Create AKS cluster
az aks create \
  --resource-group myResourceGroup \
  --name my-cluster \
  --node-count 3 \
  --node-vm-size Standard_B2s \
  --enable-addons monitoring

# Get credentials
az aks get-credentials --resource-group myResourceGroup --name my-cluster

# Verify
kubectl get nodes
```

## 🐧 Method 3: kubeadm (Production)

### Master Node Setup
```bash
# Install kubeadm, kubelet, kubectl
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

# Add Kubernetes repository
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Install
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

# Initialize cluster
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

# Setup kubeconfig
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Install network plugin (Calico)
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

### Worker Node Setup
```bash
# Install kubeadm, kubelet, kubectl (same as master)

# Join cluster (use token from master)
sudo kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

## 📦 Method 4: Docker Desktop (macOS/Windows)

### Installation
```bash
# Download Docker Desktop
# https://www.docker.com/products/docker-desktop

# Enable Kubernetes in Docker Desktop
# Settings > Kubernetes > Enable Kubernetes

# Verify
kubectl get nodes
kubectl cluster-info
```

## 📋 Prerequisites

### System Requirements
- **CPU**: 2+ cores (4+ recommended)
- **Memory**: 2GB+ RAM (4GB+ recommended)
- **Storage**: 20GB+ free space
- **Network**: Internet access
- **Operating System**: Linux, macOS, or Windows

### Required Tools
```bash
# Install kubectl
# macOS
brew install kubectl

# Linux
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Verify kubectl
kubectl version --client
```

## ⚙️ Installation Steps

### Step 1: Install kubectl
```bash
# Choose installation method based on OS
# macOS: brew install kubectl
# Linux: Follow kubectl installation guide
# Windows: Download from kubernetes.io
```

### Step 2: Choose Installation Method
```bash
# For local development: Minikube or Docker Desktop
# For production: Cloud services (EKS, GKE, AKS) or kubeadm
```

### Step 3: Set Up Cluster
```bash
# Follow method-specific instructions above
# Example for Minikube:
minikube start
```

### Step 4: Verify Installation
```bash
# Check cluster connection
kubectl cluster-info

# Check nodes
kubectl get nodes

# Check all resources
kubectl get all --all-namespaces
```

## ✅ Verification

### Test Kubernetes Installation
```bash
# Check kubectl version
kubectl version --client

# Check cluster version
kubectl version

# Check nodes
kubectl get nodes

# Check cluster info
kubectl cluster-info

# Test deployment
kubectl run nginx --image=nginx
kubectl get pods
kubectl delete pod nginx
```

### Verify Cluster Components
```bash
# Check system pods
kubectl get pods -n kube-system

# Check services
kubectl get svc -n kube-system

# Check cluster status
kubectl get componentstatuses
```

## 🔧 Post-Installation Configuration

### Configure kubectl Context
```bash
# View contexts
kubectl config get-contexts

# Switch context
kubectl config use-context my-context

# Set default namespace
kubectl config set-context --current --namespace=production
```

### Install Add-ons
```bash
# Install Kubernetes Dashboard
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

# Access dashboard
kubectl proxy
# Open http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

# Install metrics server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

## 🐛 Troubleshooting

### Issue 1: kubectl Connection Refused
```bash
# Problem: Cannot connect to cluster
# Solution: Check kubeconfig
kubectl config view

# Verify cluster endpoint
kubectl cluster-info
```

### Issue 2: Nodes Not Ready
```bash
# Problem: Nodes showing NotReady status
# Solution: Check node status
kubectl describe node <node-name>

# Check kubelet status
sudo systemctl status kubelet
```

### Issue 3: Pods Not Starting
```bash
# Problem: Pods stuck in Pending
# Solution: Check pod events
kubectl describe pod <pod-name>

# Check node resources
kubectl top nodes
```

## 📚 Next Steps

After installation:
1. **Learn Basics**: Start with pods and deployments
2. **Set Up Namespaces**: Organize resources
3. **Install Add-ons**: Dashboard, metrics server
4. **Configure Storage**: Set up persistent volumes
5. **Set Up Networking**: Configure services and ingress

---

*Kubernetes is now ready for container orchestration! 🎉*