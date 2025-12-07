# Kubernetes — Usage

## 🚀 Getting Started with Kubernetes

This guide covers practical usage examples for Kubernetes in real-world container orchestration scenarios.

## 📊 Example 1: Deploy Web Application

### Scenario: Deploy a Simple Web Application

Deploy a web application with deployment, service, and ingress.

### Step 1: Create Deployment
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
        version: v1
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
          name: http
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
        env:
        - name: ENVIRONMENT
          value: "production"
```

### Step 2: Create Service
```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-service
  labels:
    app: web-app
spec:
  type: LoadBalancer
  selector:
    app: web-app
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP
```

### Step 3: Deploy
```bash
# Apply deployment
kubectl apply -f deployment.yaml

# Apply service
kubectl apply -f service.yaml

# Check status
kubectl get deployments
kubectl get pods
kubectl get services

# View pod logs
kubectl logs -l app=web-app

# Describe deployment
kubectl describe deployment web-app
```

## 🔧 Example 2: Multi-tier Application

### Scenario: Deploy Frontend, Backend, and Database

Deploy a complete application stack with multiple components.

### Step 1: Create Namespace
```bash
# Create namespace
kubectl create namespace production

# Set context namespace
kubectl config set-context --current --namespace=production
```

### Step 2: Deploy Database
```yaml
# database.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:14
        env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
      volumes:
      - name: postgres-storage
        persistentVolumeClaim:
          claimName: postgres-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
```

### Step 3: Deploy Backend
```yaml
# backend.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: my-backend:latest
        env:
        - name: DATABASE_URL
          value: "postgresql://postgres:password@postgres-service:5432/mydb"
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
  - port: 8080
    targetPort: 8080
```

### Step 4: Deploy Frontend
```yaml
# frontend.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: my-frontend:latest
        env:
        - name: API_URL
          value: "http://backend-service:8080"
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 80
```

### Step 5: Deploy All Components
```bash
# Deploy all components
kubectl apply -f database.yaml
kubectl apply -f backend.yaml
kubectl apply -f frontend.yaml

# Check status
kubectl get all
```

## 🚀 Example 3: Auto-scaling

### Scenario: Configure Horizontal Pod Autoscaler

Set up automatic scaling based on CPU and memory usage.

### Step 1: Install Metrics Server
```bash
# Install metrics server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Verify
kubectl get deployment metrics-server -n kube-system
```

### Step 2: Create HPA
```yaml
# hpa.yaml
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
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
      - type: Pods
        value: 2
        periodSeconds: 15
      selectPolicy: Max
```

### Step 3: Apply HPA
```bash
# Apply HPA
kubectl apply -f hpa.yaml

# Check HPA status
kubectl get hpa

# Describe HPA
kubectl describe hpa web-app-hpa
```

## 🎯 Example 4: Rolling Updates

### Scenario: Update Application with Zero Downtime

Perform a rolling update of the application.

### Step 1: Update Deployment
```yaml
# Update image version
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
        version: v2  # Updated version
    spec:
      containers:
      - name: nginx
        image: nginx:1.22  # Updated image
        ports:
        - containerPort: 80
```

### Step 2: Apply Update
```bash
# Apply update
kubectl apply -f deployment.yaml

# Watch rollout
kubectl rollout status deployment/web-app

# View rollout history
kubectl rollout history deployment/web-app

# Rollback if needed
kubectl rollout undo deployment/web-app

# Rollback to specific revision
kubectl rollout undo deployment/web-app --to-revision=2
```

## 🔄 Example 5: ConfigMap and Secrets

### Scenario: Manage Configuration and Secrets

Use ConfigMaps and Secrets for application configuration.

### Step 1: Create ConfigMap
```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  app.properties: |
    database.host=postgres-service
    database.port=5432
    api.timeout=30
  nginx.conf: |
    server {
      listen 80;
      location / {
        proxy_pass http://backend-service:8080;
      }
    }
```

### Step 2: Create Secret
```bash
# Create secret from command line
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=secretpassword

# Or from file
kubectl create secret generic db-secret \
  --from-file=username=./username.txt \
  --from-file=password=./password.txt
```

### Step 3: Use in Deployment
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  template:
    spec:
      containers:
      - name: app
        image: my-app:latest
        env:
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        volumeMounts:
        - name: config
          mountPath: /etc/config
      volumes:
      - name: config
        configMap:
          name: app-config
```

## 📊 Monitoring and Debugging

### Check Resource Status
```bash
# Get all resources
kubectl get all

# Get pods with details
kubectl get pods -o wide

# Describe resource
kubectl describe pod <pod-name>

# View logs
kubectl logs <pod-name>
kubectl logs -f <pod-name>  # Follow logs

# View logs from all pods with label
kubectl logs -l app=web-app
```

### Debug Issues
```bash
# Execute command in pod
kubectl exec -it <pod-name> -- /bin/bash

# Check events
kubectl get events --sort-by='.lastTimestamp'

# Check resource usage
kubectl top nodes
kubectl top pods
```

## 🎯 Common Usage Patterns

### 1. **Deployment Workflow**
```bash
# 1. Create deployment
kubectl apply -f deployment.yaml

# 2. Check status
kubectl get deployments
kubectl get pods

# 3. View logs
kubectl logs -l app=web-app

# 4. Scale deployment
kubectl scale deployment web-app --replicas=5

# 5. Update deployment
kubectl set image deployment/web-app nginx=nginx:1.22

# 6. Rollback if needed
kubectl rollout undo deployment/web-app
```

### 2. **Service Management**
```bash
# 1. Create service
kubectl apply -f service.yaml

# 2. Check service
kubectl get services
kubectl describe service web-app-service

# 3. Port forward for testing
kubectl port-forward service/web-app-service 8080:80
```

### 3. **Namespace Management**
```bash
# 1. Create namespace
kubectl create namespace production

# 2. Set default namespace
kubectl config set-context --current --namespace=production

# 3. Deploy to namespace
kubectl apply -f deployment.yaml -n production

# 4. List resources in namespace
kubectl get all -n production
```

---

*Kubernetes is now integrated into your container orchestration workflow! 🎉*