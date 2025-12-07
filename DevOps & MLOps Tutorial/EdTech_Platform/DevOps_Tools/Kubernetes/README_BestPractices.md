# Kubernetes — Best Practices

## 🎯 Production Best Practices

### 1. **Resource Management**
```yaml
# Always set resource requests and limits
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: app
    image: my-app:latest
    resources:
      requests:
        memory: "256Mi"
        cpu: "250m"
      limits:
        memory: "512Mi"
        cpu: "500m"
```

### 2. **Health Checks**
```yaml
# Configure liveness and readiness probes
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: app
    image: my-app:latest
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 10
      timeoutSeconds: 5
      failureThreshold: 3
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
      timeoutSeconds: 3
      failureThreshold: 3
```

### 3. **Namespace Organization**
```yaml
# Use namespaces to organize resources
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    name: production
    environment: prod
```

## 🔐 Security Best Practices

### 1. **RBAC Configuration**
```yaml
# Use Role-Based Access Control
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### 2. **Secrets Management**
```yaml
# Use Kubernetes secrets (not ConfigMaps)
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
stringData:
  username: admin
  password: password

# Reference in deployment
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      containers:
      - name: app
        env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
```

### 3. **Network Policies**
```yaml
# Implement network policies
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: web-app-policy
spec:
  podSelector:
    matchLabels:
      app: web-app
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: frontend
    ports:
    - protocol: TCP
      port: 80
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: backend
    ports:
    - protocol: TCP
      port: 5432
```

### 4. **Pod Security Standards**
```yaml
# Use Pod Security Standards
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

## 📊 Deployment Best Practices

### 1. **Rolling Updates**
```yaml
# Configure rolling update strategy
apiVersion: apps/v1
kind: Deployment
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```

### 2. **Pod Disruption Budgets**
```yaml
# Set up Pod Disruption Budgets
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-app-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: web-app
```

### 3. **Resource Quotas**
```yaml
# Set resource quotas
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    persistentvolumeclaims: "10"
    pods: "10"
```

## 🔄 Auto-scaling Best Practices

### 1. **Horizontal Pod Autoscaler**
```yaml
# Configure HPA properly
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
    scaleUp:
      stabilizationWindowSeconds: 0
```

### 2. **Vertical Pod Autoscaler**
```yaml
# Use VPA for right-sizing
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: web-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  updatePolicy:
    updateMode: "Auto"
```

## 📈 Monitoring Best Practices

### 1. **Labels and Annotations**
```yaml
# Use consistent labels
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: web-app
    version: v1
    environment: production
    team: platform
  annotations:
    description: "Web application deployment"
    contact: "team@example.com"
```

### 2. **Logging**
```yaml
# Configure proper logging
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: app
    image: my-app:latest
    # Logs should go to stdout/stderr
    # Use sidecar for log aggregation if needed
```

## 🔧 Configuration Management Best Practices

### 1. **ConfigMaps**
```yaml
# Use ConfigMaps for non-sensitive config
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  app.properties: |
    database.host=postgres-service
    api.timeout=30
```

### 2. **Secrets**
```yaml
# Use Secrets for sensitive data
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
stringData:
  password: secretpassword
```

## 🚨 Common Pitfalls to Avoid

### 1. **Don't Use Default Namespace**
```yaml
# Bad: Use default namespace
# Good: Create dedicated namespaces
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

### 2. **Don't Skip Resource Limits**
```yaml
# Bad: No resource limits
spec:
  containers:
  - name: app
    image: my-app:latest

# Good: Set resource limits
spec:
  containers:
  - name: app
    image: my-app:latest
    resources:
      requests:
        memory: "256Mi"
        cpu: "250m"
      limits:
        memory: "512Mi"
        cpu: "500m"
```

### 3. **Don't Hardcode Secrets**
```yaml
# Bad: Hardcoded secrets
env:
- name: PASSWORD
  value: "secret123"

# Good: Use Secrets
env:
- name: PASSWORD
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: password
```

### 4. **Don't Ignore Health Checks**
```yaml
# Bad: No health checks
spec:
  containers:
  - name: app
    image: my-app:latest

# Good: Configure health checks
spec:
  containers:
  - name: app
    image: my-app:latest
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
```

---

*Follow these best practices for production Kubernetes deployments! 🎯*