# ArgoCD — Usage

## 🚀 Getting Started with ArgoCD

This guide covers practical usage examples for ArgoCD in real-world GitOps scenarios.

## 📊 Example 1: Deploy Application from Git

### Scenario: Deploy a Simple Application

Deploy a web application from a Git repository to Kubernetes.

### Step 1: Prepare Application Manifests
```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app
        image: my-app:1.0.0
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 8080
```

### Step 2: Create ArgoCD Application
```yaml
# application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/my-app
    targetRevision: main
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

### Step 3: Apply Application
```bash
# Apply application
kubectl apply -f application.yaml

# Check application status
argocd app get my-app

# Or via UI
# ArgoCD UI > Applications > my-app
```

## 🔧 Example 2: Multi-environment Deployment

### Scenario: Deploy to Multiple Environments

Deploy the same application to development, staging, and production.

### Step 1: Create Environment-specific Applications
```yaml
# applications-dev.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-dev
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/my-app
    targetRevision: develop
    path: k8s/overlays/dev
  destination:
    server: https://kubernetes.default.svc
    namespace: dev
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
---
# applications-staging.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-staging
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/my-app
    targetRevision: main
    path: k8s/overlays/staging
  destination:
    server: https://kubernetes.default.svc
    namespace: staging
  syncPolicy:
    automated:
      prune: false
      selfHeal: false
---
# applications-prod.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-prod
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/my-app
    targetRevision: main
    path: k8s/overlays/prod
  destination:
    server: https://kubernetes.default.svc
    namespace: prod
  syncPolicy:
    automated:
      prune: false
      selfHeal: false
```

### Step 2: Apply Applications
```bash
# Apply all applications
kubectl apply -f applications-dev.yaml
kubectl apply -f applications-staging.yaml
kubectl apply -f applications-prod.yaml

# Check status
argocd app list
```

## 🚀 Example 3: Helm Chart Deployment

### Scenario: Deploy Helm Chart

Deploy an application using Helm charts.

### Step 1: Create Application for Helm
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-helm-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://charts.example.com
    chart: my-chart
    targetRevision: 1.0.0
    helm:
      releaseName: my-release
      values: |
        replicaCount: 3
        image:
          repository: my-app
          tag: 1.0.0
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### Step 2: Deploy
```bash
# Apply application
kubectl apply -f helm-application.yaml

# Check status
argocd app get my-helm-app
```

## 🎯 Example 4: Application of Applications (App of Apps)

### Scenario: Manage Multiple Applications

Use the App of Apps pattern to manage multiple applications.

### Step 1: Create Root Application
```yaml
# root-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: root-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/apps
    targetRevision: main
    path: applications
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### Step 2: Create Child Applications
```yaml
# applications/app1.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: app1
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/app1
    targetRevision: main
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: app1
---
# applications/app2.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: app2
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/app2
    targetRevision: main
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: app2
```

## 🔄 Example 5: Sync and Rollback

### Scenario: Manual Sync and Rollback

Manually sync applications and rollback when needed.

### Step 1: Manual Sync
```bash
# Sync application
argocd app sync my-app

# Sync with specific revision
argocd app sync my-app --revision v1.0.0

# Sync with prune
argocd app sync my-app --prune
```

### Step 2: Rollback
```bash
# View application history
argocd app history my-app

# Rollback to previous version
argocd app rollback my-app

# Rollback to specific revision
argocd app rollback my-app <revision-id>
```

## 📊 Monitoring and Debugging

### Check Application Status
```bash
# List all applications
argocd app list

# Get application details
argocd app get my-app

# View application logs
argocd app logs my-app

# View application events
kubectl get events -n argocd --field-selector involvedObject.name=my-app
```

### Debug Sync Issues
```bash
# Check application sync status
argocd app get my-app

# View diff
argocd app diff my-app

# Check application health
argocd app health my-app
```

## 🎯 Common Usage Patterns

### 1. **GitOps Workflow**
```bash
# 1. Update application manifests in Git
# 2. Commit and push changes
# 3. ArgoCD automatically syncs (if automated)
# 4. Or manually sync: argocd app sync my-app
```

### 2. **Multi-environment Management**
```bash
# 1. Create separate applications per environment
# 2. Use different Git branches/paths
# 3. Configure environment-specific sync policies
# 4. Monitor all environments from ArgoCD UI
```

### 3. **Application Rollback**
```bash
# 1. View application history
argocd app history my-app

# 2. Rollback to previous version
argocd app rollback my-app

# 3. Verify rollback
argocd app get my-app
```

---

*ArgoCD is now integrated into your GitOps workflow! 🎉*