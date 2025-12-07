# Kubeflow — Installation

## 🚀 Installation Methods

Kubeflow can be installed on various Kubernetes distributions and cloud providers.

## ☁️ Method 1: Cloud Installation (Recommended)

### Google Cloud Platform (GCP)
```bash
# Install gcloud CLI
curl https://sdk.cloud.google.com | bash

# Set project
export PROJECT_ID=your-project-id
gcloud config set project $PROJECT_ID

# Enable required APIs
gcloud services enable container.googleapis.com

# Create GKE cluster
gcloud container clusters create kubeflow-cluster \
  --zone=us-central1-a \
  --machine-type=n1-standard-4 \
  --num-nodes=3 \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=10

# Install Kubeflow
kubectl apply -k "github.com/kubeflow/manifests/example?ref=v1.7.0"
```

### Amazon Web Services (AWS)
```bash
# Install eksctl
export RELEASE=0.140.0
curl -sSL "https://github.com/weaveworks/eksctl/releases/download/v${RELEASE}/eksctl_$(uname -s)_amd64.tar.gz" | tar -xz
sudo mv eksctl /usr/local/bin

# Create EKS cluster
eksctl create cluster --name kubeflow-cluster --region us-west-2

# Install Kubeflow
kubectl apply -k "github.com/kubeflow/manifests/example?ref=v1.7.0"
```

### Microsoft Azure
```bash
# Install Azure CLI
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Login to Azure
az login

# Create AKS cluster
az aks create --resource-group myResourceGroup --name kubeflow-cluster --node-count 3

# Get credentials
az aks get-credentials --resource-group myResourceGroup --name kubeflow-cluster

# Install Kubeflow
kubectl apply -k "github.com/kubeflow/manifests/example?ref=v1.7.0"
```

## 🏠 Method 2: Local Installation

### Minikube (Development)
```bash
# Install Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start Minikube with sufficient resources
minikube start --cpus=4 --memory=8192 --disk-size=50g

# Install Kubeflow
kubectl apply -k "github.com/kubeflow/manifests/example?ref=v1.7.0"
```

### Kind (Kubernetes in Docker)
```bash
# Install Kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Create Kind cluster
kind create cluster --name kubeflow --config kind-config.yaml

# Install Kubeflow
kubectl apply -k "github.com/kubeflow/manifests/example?ref=v1.7.0"
```

## 📋 Prerequisites

### System Requirements
- **Kubernetes**: 1.21+ (recommended 1.24+)
- **CPU**: 4+ cores
- **Memory**: 8GB+ RAM
- **Storage**: 50GB+ free space
- **Network**: Internet access for image pulls

### Required Tools
```bash
# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Install kustomize
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash
sudo mv kustomize /usr/local/bin/
```

## ⚙️ Installation Steps

### Step 1: Verify Kubernetes Cluster
```bash
# Check cluster status
kubectl cluster-info
kubectl get nodes

# Verify cluster resources
kubectl top nodes
```

### Step 2: Install Kubeflow
```bash
# Clone Kubeflow manifests
git clone https://github.com/kubeflow/manifests.git
cd manifests

# Install Kubeflow (this may take 10-15 minutes)
kubectl apply -k example/

# Wait for installation to complete
kubectl wait --for=condition=ready pod -l app=centraldashboard -n kubeflow --timeout=600s
```

### Step 3: Access Kubeflow Dashboard
```bash
# Port forward to access dashboard
kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80

# Access dashboard at http://localhost:8080
```

## ✅ Verification

### Check Installation Status
```bash
# Check all pods are running
kubectl get pods -n kubeflow

# Check services
kubectl get svc -n kubeflow

# Check ingress
kubectl get ingress -n kubeflow
```

### Test Basic Functionality
```bash
# Create a simple pipeline
kubectl apply -f - <<EOF
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  name: test-pipeline
  namespace: kubeflow
spec:
  entrypoint: hello
  templates:
  - name: hello
    container:
      image: alpine:latest
      command: [echo, "Hello Kubeflow!"]
EOF

# Check pipeline status
kubectl get workflow -n kubeflow
```

## 🔧 Post-Installation Configuration

### Configure Storage
```bash
# Create storage class for persistent volumes
kubectl apply -f - <<EOF
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
  replication-type: regional-pd
EOF
```

### Configure Authentication
```bash
# Enable authentication (example with Dex)
kubectl patch configmap dex -n auth --patch '{
  "data": {
    "config.yaml": "issuer: http://dex.auth.svc.cluster.local:5556/dex"
  }
}'
```

## 🐛 Common Installation Issues

### Issue 1: Insufficient Resources
```bash
# Error: Pod pending due to insufficient resources
# Solution: Increase cluster resources
minikube start --cpus=6 --memory=12288 --disk-size=100g
```

### Issue 2: Image Pull Errors
```bash
# Error: ImagePullBackOff
# Solution: Check image registry access
kubectl describe pod <pod-name> -n kubeflow

# Pre-pull images if needed
kubectl get pods -n kubeflow -o jsonpath='{.items[*].spec.containers[*].image}' | tr -s '[[:space:]]' '\n' | sort | uniq
```

### Issue 3: Network Issues
```bash
# Error: Service not accessible
# Solution: Check network policies
kubectl get networkpolicies -n kubeflow
kubectl get svc -n istio-system
```

### Issue 4: Persistent Volume Issues
```bash
# Error: Pod pending due to PVC
# Solution: Check storage class
kubectl get storageclass
kubectl get pv
kubectl get pvc -n kubeflow
```

## 🔍 Troubleshooting Commands

### Check Component Status
```bash
# Check all Kubeflow components
kubectl get pods -n kubeflow
kubectl get pods -n istio-system
kubectl get pods -n knative-serving

# Check logs for failing components
kubectl logs -n kubeflow deployment/centraldashboard
kubectl logs -n kubeflow deployment/ml-pipeline
```

### Monitor Installation Progress
```bash
# Watch installation progress
kubectl get pods -n kubeflow -w

# Check events
kubectl get events -n kubeflow --sort-by=.metadata.creationTimestamp
```

## 🚀 Next Steps

After successful installation:

1. **📖 Read**: [Configuration Guide](README_Configuration.md)
2. **🛠️ Setup**: Configure your first ML pipeline
3. **📊 Practice**: Create a Jupyter notebook server
4. **🔄 Learn**: Build your first Kubeflow pipeline

---

*Kubeflow is now ready for your ML workflows! 🎉*
