# Istio — Configuration

## ⚙️ Configuration Overview

Istio configuration is managed through Custom Resource Definitions (CRDs) and ConfigMaps.

## 📁 Configuration Files

### 1. **VirtualService**
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
    - uri:
        prefix: "/api"
    route:
    - destination:
        host: my-app
        subset: v1
      weight: 90
    - destination:
        host: my-app
        subset: v2
      weight: 10
```

### 2. **DestinationRule**
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
  trafficPolicy:
    loadBalancer:
      simple: ROUND_ROBIN
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 10
        http2MaxRequests: 2
        maxRequestsPerConnection: 1
```

### 3. **Gateway**
```yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: my-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - example.com
```

## 🔧 Basic Configuration

### 1. **Sidecar Injection**
```yaml
# Enable automatic sidecar injection
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
  labels:
    istio-injection: enabled
```

### 2. **Service Entry**
```yaml
apiVersion: networking.istio.io/v1beta1
kind: ServiceEntry
metadata:
  name: external-service
spec:
  hosts:
  - api.example.com
  ports:
  - number: 443
    name: https
    protocol: HTTPS
  location: MESH_EXTERNAL
  resolution: DNS
```

## 🔐 Security Configuration

### 1. **PeerAuthentication**
```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
spec:
  mtls:
    mode: STRICT
```

### 2. **AuthorizationPolicy**
```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-get
spec:
  selector:
    matchLabels:
      app: my-app
  action: ALLOW
  rules:
  - to:
    - operation:
        methods: ["GET"]
```

---

*Istio configuration optimized! 🎯*