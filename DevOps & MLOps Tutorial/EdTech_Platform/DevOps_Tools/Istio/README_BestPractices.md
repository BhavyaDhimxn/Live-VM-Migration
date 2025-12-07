# Istio — Best Practices

## 🎯 Production Best Practices

### 1. **Service Mesh Organization**
```
istio-config/
├── virtualservices/
│   ├── app1-vs.yaml
│   └── app2-vs.yaml
├── destinationrules/
│   ├── app1-dr.yaml
│   └── app2-dr.yaml
├── gateways/
│   └── ingress-gateway.yaml
└── security/
    ├── peer-authentication.yaml
    └── authorization-policies.yaml
```

### 2. **Resource Naming Convention**
```yaml
# Consistent resource naming
# Format: {service-name}-{resource-type}
# Examples:
# - my-app-virtualservice
# - my-app-destinationrule
# - my-app-gateway
```

## 🔐 Security Best Practices

### 1. **mTLS Configuration**
```yaml
# Enable mTLS for all services
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: default
spec:
  mtls:
    mode: STRICT  # Enforce mTLS
```

### 2. **Authorization Policies**
```yaml
# Implement least privilege
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-specific
spec:
  selector:
    matchLabels:
      app: my-app
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/default/sa/my-service"]
    to:
    - operation:
        methods: ["GET", "POST"]
```

### 3. **Network Policies**
```yaml
# Combine with Kubernetes NetworkPolicies
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-istio
spec:
  podSelector:
    matchLabels:
      app: my-app
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: istio-system
```

## 📊 Traffic Management Best Practices

### 1. **VirtualService Design**
```yaml
# Use clear routing rules
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
        prefix: "/api/v1"
    route:
    - destination:
        host: my-app
        subset: v1
  - match:
    - uri:
        prefix: "/api/v2"
    route:
    - destination:
        host: my-app
        subset: v2
```

### 2. **DestinationRule Configuration**
```yaml
# Configure appropriate load balancing
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: my-app
spec:
  host: my-app
  trafficPolicy:
    loadBalancer:
      simple: LEAST_CONN  # Use appropriate algorithm
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 10
```

### 3. **Circuit Breaker Configuration**
```yaml
# Implement circuit breakers
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: my-app
spec:
  host: my-app
  trafficPolicy:
    outlierDetection:
      consecutiveErrors: 3
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
```

## 🔄 Resilience Best Practices

### 1. **Retry Configuration**
```yaml
# Configure retries
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
    retries:
      attempts: 3
      perTryTimeout: 2s
      retryOn: 5xx,reset,connect-failure
```

### 2. **Timeout Configuration**
```yaml
# Set appropriate timeouts
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
    timeout: 30s
```

## 📈 Performance Optimization

### 1. **Sidecar Configuration**
```yaml
# Optimize sidecar resources
apiVersion: v1
kind: Pod
metadata:
  annotations:
    sidecar.istio.io/proxyCPU: "100m"
    sidecar.istio.io/proxyMemory: "128Mi"
spec:
  containers:
  - name: app
    image: my-app:latest
```

### 2. **Connection Pooling**
```yaml
# Configure connection pools
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
```

## 🔧 Reproducibility Best Practices

### 1. **Configuration Versioning**
```bash
# Version control Istio configurations
git init
git add virtualservices/ destinationrules/
git commit -m "Add Istio configurations"
```

### 2. **Environment-specific Configurations**
```yaml
# Use environment-specific configurations
# development/
#   - virtualservice-dev.yaml
# production/
#   - virtualservice-prod.yaml
```

## 🧪 Testing Best Practices

### 1. **Traffic Testing**
```bash
# Test traffic routing
curl -H "Host: my-app" http://ingress-gateway/api

# Test with different headers
curl -H "Host: my-app" -H "user-agent: Mobile" http://ingress-gateway/api
```

### 2. **Security Testing**
```bash
# Test mTLS
istioctl authn tls-check <pod-name> <service-name>

# Test authorization
curl -v http://service-name/api
```

## 📚 Documentation Best Practices

### 1. **Configuration Documentation**
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my-app
  annotations:
    description: "Routes traffic to my-app service"
    team: "platform-team"
spec:
  # ...
```

### 2. **Policy Documentation**
```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: my-app-policy
  annotations:
    description: "Authorization policy for my-app"
spec:
  # ...
```

## 🚨 Common Pitfalls to Avoid

### 1. **Don't Over-configure**
```yaml
# Bad: Too many routing rules
http:
  - match: [/* many matches */]
    route: [/* many routes */]

# Good: Clear, focused routing
http:
  - match:
    - uri:
        prefix: "/api"
    route:
    - destination:
        host: my-app
```

### 2. **Don't Ignore mTLS**
```yaml
# Bad: Disabled mTLS
spec:
  mtls:
    mode: DISABLE

# Good: Enforce mTLS
spec:
  mtls:
    mode: STRICT
```

### 3. **Don't Skip Circuit Breakers**
```yaml
# Bad: No circuit breaker
spec:
  trafficPolicy: {}

# Good: Configure circuit breaker
spec:
  trafficPolicy:
    outlierDetection:
      consecutiveErrors: 3
```

---

*Follow these best practices to build effective service mesh with Istio! 🎯*