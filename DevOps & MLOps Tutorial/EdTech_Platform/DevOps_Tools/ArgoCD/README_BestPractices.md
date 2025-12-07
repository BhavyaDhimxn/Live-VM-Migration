# ArgoCD — Best Practices

## 🎯 Production Best Practices

### 1. **Application Organization**
```
argocd-apps/
├── applications/
│   ├── app-of-apps.yaml
│   ├── app1.yaml
│   └── app2.yaml
├── projects/
│   └── my-project.yaml
└── repositories/
    └── repo-credentials.yaml
```

### 2. **Application Naming Convention**
```yaml
# Consistent application naming
# Format: {service}-{environment}
# Examples:
# - web-app-dev
# - api-service-prod
# - database-staging
```

## 🔐 Security Best Practices

### 1. **RBAC Configuration**
```yaml
# Configure RBAC policies
# argocd-rbac-cm
data:
  policy.csv: |
    p, role:admin, applications, *, */*, allow
    p, role:admin, clusters, *, *, allow
    p, role:admin, repositories, *, *, allow
    p, role:developer, applications, get, */*, allow
    p, role:developer, applications, sync, */*, allow
    g, admin, role:admin
    g, developer, role:developer
```

### 2. **Repository Access**
```yaml
# Use repository credentials
apiVersion: v1
kind: Secret
metadata:
  name: repo-credentials
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repository
type: Opaque
stringData:
  type: git
  url: https://github.com/example/repo
  password: $GIT_TOKEN
  username: git
```

### 3. **OIDC Authentication**
```yaml
# Use OIDC for authentication
# argocd-cm
data:
  oidc.config: |
    name: Okta
    issuer: https://dev-123456.okta.com
    clientId: abc123
    requestedScopes: ["openid", "profile", "email", "groups"]
```

## 📊 Application Best Practices

### 1. **Sync Policy Configuration**
```yaml
# Use appropriate sync policies
spec:
  syncPolicy:
    automated:
      prune: true  # Prune resources not in Git
      selfHeal: true  # Auto-heal when drift detected
    syncOptions:
      - CreateNamespace=true  # Create namespace if missing
      - PrunePropagationPolicy=foreground  # Prune policy
```

### 2. **Health Checks**
```yaml
# Configure health checks
spec:
  syncPolicy:
    syncOptions:
      - RespectIgnoreDifferences=true
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas
```

### 3. **Resource Limits**
```yaml
# Set resource limits in projects
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: my-project
spec:
  sourceRepos:
    - 'https://github.com/example/*'
  destinations:
    - namespace: 'default'
      server: 'https://kubernetes.default.svc'
  clusterResourceWhitelist:
    - group: ''
      kind: Namespace
```

## 🔄 CI/CD Integration

### 1. **Automated Sync**
```yaml
# Enable automated sync for development
spec:
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### 2. **Manual Sync for Production**
```yaml
# Disable automated sync for production
spec:
  syncPolicy:
    automated: null
    syncOptions:
      - CreateNamespace=true
```

### 3. **GitHub Actions Integration**
```yaml
# .github/workflows/deploy.yml
name: Deploy to ArgoCD

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    
    - name: Sync ArgoCD application
      run: |
        argocd app sync my-app --server $ARGOCD_SERVER --auth-token $ARGOCD_TOKEN
```

## 📈 Monitoring Best Practices

### 1. **Application Health Monitoring**
```bash
# Monitor application health
argocd app get my-app

# Check application sync status
argocd app sync-status my-app

# View application events
kubectl get events -n argocd --field-selector involvedObject.name=my-app
```

### 2. **Alert Configuration**
```yaml
# Configure alerts for application sync failures
# Use Prometheus alerts or external monitoring
```

## ⚡ Performance Optimization

### 1. **Repository Optimization**
```yaml
# Use shallow clones for large repositories
spec:
  source:
    repoURL: https://github.com/example/repo
    targetRevision: main
    path: k8s
```

### 2. **Sync Optimization**
```yaml
# Use appropriate sync intervals
spec:
  syncPolicy:
    syncOptions:
      - RespectIgnoreDifferences=true
      - CreateNamespace=true
```

## 🔧 Reproducibility Best Practices

### 1. **Version Control**
```bash
# Version control ArgoCD applications
git init
git add applications/
git commit -m "Add ArgoCD applications"
```

### 2. **Application Versioning**
```yaml
# Use specific Git revisions
spec:
  source:
    repoURL: https://github.com/example/repo
    targetRevision: v1.0.0  # Specific version
    path: k8s
```

## 🧪 Testing Best Practices

### 1. **Application Testing**
```bash
# Test application sync
argocd app sync my-app --dry-run

# Test application diff
argocd app diff my-app

# Validate application
argocd app validate my-app
```

### 2. **Sync Testing**
```bash
# Test sync without applying
argocd app sync my-app --dry-run

# Test sync with specific revision
argocd app sync my-app --revision v1.0.0 --dry-run
```

## 📚 Documentation Best Practices

### 1. **Application Documentation**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  annotations:
    description: "Web application for user management"
    team: "platform-team"
spec:
  # ...
```

### 2. **Project Documentation**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: my-project
spec:
  description: "Project for web applications"
  # ...
```

## 🚨 Common Pitfalls to Avoid

### 1. **Don't Use Automated Sync for Production**
```yaml
# Bad: Automated sync for production
spec:
  syncPolicy:
    automated:
      prune: true

# Good: Manual sync for production
spec:
  syncPolicy:
    automated: null
```

### 2. **Don't Ignore Resource Limits**
```yaml
# Bad: No resource limits
spec:
  destination:
    namespace: '*'

# Good: Set resource limits
spec:
  destination:
    namespace: 'default'
  # Configure in AppProject
```

### 3. **Don't Hardcode Credentials**
```yaml
# Bad: Hardcoded credentials
stringData:
  password: my-password

# Good: Use secrets or environment variables
stringData:
  password: $GIT_TOKEN
```

---

*Follow these best practices to build effective GitOps workflows with ArgoCD! 🎯*