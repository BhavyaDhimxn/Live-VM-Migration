# Istio — Usage

## 🚀 Getting Started with Istio

This guide covers practical usage examples for Istio in real-world service mesh scenarios.

## 📊 Example 1: Basic Traffic Routing

### Scenario: Route Traffic to Multiple Versions

Route traffic between different versions of an application.

### Step 1: Deploy Application Versions
```yaml
# deployment-v1.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: v1
  template:
    metadata:
      labels:
        app: my-app
        version: v1
    spec:
      containers:
      - name: app
        image: my-app:v1
---
# deployment-v2.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-v2
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: v2
  template:
    metadata:
      labels:
        app: my-app
        version: v2
    spec:
      containers:
      - name: app
        image: my-app:v2
---
# service.yaml
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

### Step 2: Create DestinationRule
```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: my-app
spec:
  host: my-app
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

### Step 3: Create VirtualService
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my-app
spec:
  hosts:
  - my-app
  http:
  - route:
    - destination:
        host: my-app
        subset: v1
      weight: 90
    - destination:
        host: my-app
        subset: v2
      weight: 10
```

## 🔧 Example 2: Canary Deployment

### Scenario: Gradual Rollout

Gradually roll out a new version using canary deployment.

### Step 1: Initial Deployment
```yaml
# Start with 100% v1
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my-app
spec:
  hosts:
  - my-app
  http:
  - route:
    - destination:
        host: my-app
        subset: v1
      weight: 100
```

### Step 2: Gradual Rollout
```yaml
# 10% to v2
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my-app
spec:
  hosts:
  - my-app
  http:
  - route:
    - destination:
        host: my-app
        subset: v1
      weight: 90
    - destination:
        host: my-app
        subset: v2
      weight: 10
```

### Step 3: Complete Rollout
```yaml
# 100% to v2
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my-app
spec:
  hosts:
  - my-app
  http:
  - route:
    - destination:
        host: my-app
        subset: v2
      weight: 100
```

## 🚀 Example 3: Circuit Breaker

### Scenario: Implement Circuit Breaker

Implement circuit breaker pattern for resilience.

### Step 1: Configure DestinationRule
```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: my-app
spec:
  host: my-app
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 10
        http2MaxRequests: 2
        maxRequestsPerConnection: 1
    outlierDetection:
      consecutiveErrors: 3
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
      minHealthPercent: 50
```

## 🎯 Example 4: mTLS Security

### Scenario: Enable Mutual TLS

Enable mutual TLS for secure service-to-service communication.

### Step 1: Enable mTLS
```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: default
spec:
  mtls:
    mode: STRICT
```

### Step 2: Verify mTLS
```bash
# Check mTLS status
istioctl authn tls-check my-app-pod-xxx my-app

# View proxy configuration
istioctl proxy-config cluster my-app-pod-xxx
```

## 🔄 Example 5: Request Routing

### Scenario: Route Based on Headers

Route requests based on headers or other criteria.

### Step 1: Create VirtualService
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my-app
spec:
  hosts:
  - my-app
  http:
  - match:
    - headers:
        user-agent:
          regex: ".*Mobile.*"
    route:
    - destination:
        host: my-app
        subset: mobile
  - match:
    - headers:
        user-agent:
          regex: ".*Desktop.*"
    route:
    - destination:
        host: my-app
        subset: desktop
  - route:
    - destination:
        host: my-app
        subset: default
```

## 📊 Monitoring and Debugging

### Check Service Mesh Status
```bash
# Check proxy status
istioctl proxy-status

# Check proxy configuration
istioctl proxy-config cluster <pod-name>

# Check routes
istioctl proxy-config route <pod-name>

# Check listeners
istioctl proxy-config listener <pod-name>
```

### Debug Traffic Issues
```bash
# Enable access logs
kubectl logs <pod-name> -c istio-proxy

# Check metrics
kubectl exec <pod-name> -c istio-proxy -- pilot-agent request GET stats

# View Envoy admin interface
kubectl port-forward <pod-name> 15000:15000
# Access http://localhost:15000
```

## 🎯 Common Usage Patterns

### 1. **Traffic Management**
```yaml
# 1. Create DestinationRule
# 2. Create VirtualService
# 3. Configure routing rules
# 4. Monitor traffic distribution
```

### 2. **Security**
```yaml
# 1. Enable mTLS
# 2. Configure AuthorizationPolicy
# 3. Set up authentication
# 4. Monitor security metrics
```

### 3. **Observability**
```bash
# 1. Install addons (Prometheus, Grafana, Jaeger)
# 2. View metrics in Grafana
# 3. Trace requests in Jaeger
# 4. Monitor service mesh in Kiali
```

---

*Istio is now integrated into your service mesh! 🎉*