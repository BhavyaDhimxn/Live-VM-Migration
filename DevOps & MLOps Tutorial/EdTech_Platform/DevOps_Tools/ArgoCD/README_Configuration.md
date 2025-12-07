# ArgoCD — Configuration

## ⚙️ Configuration Overview

ArgoCD configuration is managed through ConfigMaps, Secrets, and Application CRDs.

## 📁 Configuration Files

### 1. **ArgoCD ConfigMap** (`argocd-cm`)
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  url: https://argocd.example.com
  application.instanceLabelKey: argocd.argoproj.io/instance
  admin.enabled: "true"
  oidc.config: |
    name: Okta
    issuer: https://dev-123456.okta.com
    clientId: abc123
    requestedScopes: ["openid", "profile", "email", "groups"]
```

### 2. **RBAC Configuration** (`argocd-rbac-cm`)
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
  namespace: argocd
data:
  policy.default: role:readonly
  policy.csv: |
    p, role:org-admin, applications, *, */*, allow
    p, role:org-admin, clusters, get, *, allow
    p, role:org-admin, repositories, get, *, allow
    g, org-admin, role:org-admin
```

### 3. **Repository Configuration**
```yaml
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
  url: https://github.com/example/my-repo
  password: your-token
  username: your-username
```

## 🔧 Basic Configuration

### 1. **Application Configuration**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/my-app
    targetRevision: HEAD
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

### 2. **Project Configuration**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: my-project
  namespace: argocd
spec:
  description: My project description
  sourceRepos:
    - '*'
  destinations:
    - namespace: '*'
      server: '*'
  clusterResourceWhitelist:
    - group: '*'
      kind: '*'
```

## 🔐 Security Configuration

### 1. **OIDC Authentication**
```yaml
# argocd-cm
data:
  oidc.config: |
    name: Okta
    issuer: https://dev-123456.okta.com
    clientId: abc123
    clientSecret: $oidc.clientSecret
    requestedScopes: ["openid", "profile", "email", "groups"]
```

### 2. **RBAC Policies**
```yaml
# argocd-rbac-cm
data:
  policy.csv: |
    p, role:admin, applications, *, */*, allow
    p, role:admin, clusters, *, *, allow
    p, role:admin, repositories, *, *, allow
    g, admin, role:admin
```

---

*ArgoCD configuration optimized! 🎯*