# Kubeflow — Configuration

## ⚙️ Configuration Overview

Kubeflow configuration involves setting up various components, customizing the platform for your specific needs, and integrating with external services.

## 📁 Configuration Files

### 1. **Kubeflow Configuration** (`kubeflow-config.yaml`)
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: kubeflow-config
  namespace: kubeflow
data:
  # Central dashboard configuration
  centraldashboard:
    title: "My ML Platform"
    logo: "/images/logo.png"
  
  # Jupyter configuration
  jupyter:
    defaultImage: "jupyter/tensorflow-notebook:latest"
    cpu: "1"
    memory: "2Gi"
  
  # Pipeline configuration
  pipeline:
    defaultImage: "python:3.9"
    timeout: "3600"
```

### 2. **Istio Configuration** (`istio-config.yaml`)
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: kubeflow-gateway
  namespace: kubeflow
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
```

## 🔧 Component Configuration

### 1. **Central Dashboard Configuration**
```yaml
# centraldashboard-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: centraldashboard-config
  namespace: kubeflow
data:
  config.yaml: |
    title: "My ML Platform"
    logo: "/images/logo.png"
    links:
    - text: "Documentation"
      url: "https://docs.mycompany.com"
    - text: "Support"
      url: "https://support.mycompany.com"
```

### 2. **Jupyter Hub Configuration**
```yaml
# jupyterhub-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: jupyterhub-config
  namespace: kubeflow
data:
  config.yaml: |
    hub:
      baseUrl: "/jupyter"
      cookieSecret: "your-secret-key"
    
    singleuser:
      image:
        name: jupyter/tensorflow-notebook
        tag: latest
      cpu:
        limit: 2
        guarantee: 1
      memory:
        limit: 4Gi
        guarantee: 2Gi
      storage:
        capacity: 10Gi
        type: dynamic
```

### 3. **Pipeline Configuration**
```yaml
# pipeline-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: pipeline-config
  namespace: kubeflow
data:
  config.yaml: |
    defaultImage: "python:3.9"
    timeout: "3600"
    maxConcurrentRuns: 10
    artifactRepository:
      type: "s3"
      s3:
        bucket: "my-pipeline-artifacts"
        region: "us-west-2"
```

## ☁️ Cloud Provider Configuration

### 1. **AWS Configuration**
```yaml
# aws-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-config
  namespace: kubeflow
data:
  config.yaml: |
    region: "us-west-2"
    s3:
      bucket: "my-kubeflow-artifacts"
      region: "us-west-2"
    eks:
      clusterName: "kubeflow-cluster"
      nodeGroup: "kubeflow-nodes"
```

### 2. **GCP Configuration**
```yaml
# gcp-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: gcp-config
  namespace: kubeflow
data:
  config.yaml: |
    project: "my-gcp-project"
    region: "us-central1"
    gcs:
      bucket: "my-kubeflow-artifacts"
    gke:
      clusterName: "kubeflow-cluster"
      nodePool: "kubeflow-nodes"
```

### 3. **Azure Configuration**
```yaml
# azure-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: azure-config
  namespace: kubeflow
data:
  config.yaml: |
    subscription: "my-azure-subscription"
    resourceGroup: "kubeflow-rg"
    location: "eastus"
    storage:
      account: "mykubeflowstorage"
      container: "artifacts"
    aks:
      clusterName: "kubeflow-cluster"
      nodePool: "kubeflow-nodes"
```

## 🔐 Authentication Configuration

### 1. **Dex Configuration (OIDC)**
```yaml
# dex-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: dex-config
  namespace: auth
data:
  config.yaml: |
    issuer: http://dex.auth.svc.cluster.local:5556/dex
    storage:
      type: kubernetes
      config:
        inCluster: true
    web:
      http: 0.0.0.0:5556
    connectors:
    - type: ldap
      id: ldap
      name: LDAP
      config:
        host: ldap.company.com:636
        insecureNoSSL: false
        bindDN: cn=admin,dc=company,dc=com
        bindPW: password
        usernamePrompt: Email Address
        userSearch:
          baseDN: ou=Users,dc=company,dc=com
          filter: "(objectClass=person)"
          username: mail
          idAttr: DN
          emailAttr: mail
          nameAttr: cn
```

### 2. **RBAC Configuration**
```yaml
# rbac-config.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: kubeflow-user
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubeflow-user-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kubeflow-user
subjects:
- kind: User
  name: user@company.com
  apiGroup: rbac.authorization.k8s.io
```

## 📊 Storage Configuration

### 1. **Persistent Volume Configuration**
```yaml
# pv-config.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: kubeflow-pv
spec:
  capacity:
    storage: 100Gi
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: fast-ssd
  nfs:
    server: nfs-server.company.com
    path: /kubeflow
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: kubeflow-pvc
  namespace: kubeflow
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 100Gi
  storageClassName: fast-ssd
```

### 2. **Storage Class Configuration**
```yaml
# storage-class.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
  replication-type: regional-pd
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```

## 🔄 Pipeline Configuration

### 1. **Pipeline Defaults**
```yaml
# pipeline-defaults.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: pipeline-defaults
  namespace: kubeflow
data:
  config.yaml: |
    defaultImage: "python:3.9"
    timeout: "3600"
    maxConcurrentRuns: 10
    artifactRepository:
      type: "s3"
      s3:
        bucket: "my-pipeline-artifacts"
        region: "us-west-2"
        keyFormat: "artifacts/{{workflow.uid}}/{{pod.name}}"
```

### 2. **Pipeline Templates**
```yaml
# pipeline-templates.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: pipeline-templates
  namespace: kubeflow
data:
  templates.yaml: |
    templates:
    - name: "data-preparation"
      image: "python:3.9"
      command: ["python", "prepare.py"]
      resources:
        requests:
          cpu: "1"
          memory: "2Gi"
        limits:
          cpu: "2"
          memory: "4Gi"
    
    - name: "model-training"
      image: "tensorflow/tensorflow:latest"
      command: ["python", "train.py"]
      resources:
        requests:
          cpu: "2"
          memory: "4Gi"
          nvidia.com/gpu: "1"
        limits:
          cpu: "4"
          memory: "8Gi"
          nvidia.com/gpu: "1"
```

## 📈 Monitoring Configuration

### 1. **Prometheus Configuration**
```yaml
# prometheus-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
      evaluation_interval: 15s
    
    scrape_configs:
    - job_name: 'kubeflow'
      static_configs:
      - targets: ['centraldashboard.kubeflow.svc.cluster.local:8080']
    
    - job_name: 'kubernetes-pods'
      kubernetes_sd_configs:
      - role: pod
      relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
```

### 2. **Grafana Configuration**
```yaml
# grafana-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-config
  namespace: monitoring
data:
  grafana.ini: |
    [server]
    root_url = http://grafana.monitoring.svc.cluster.local:3000/
    
    [auth.anonymous]
    enabled = true
    org_name = Main Org.
    org_role = Viewer
    
    [dashboards]
    default_home_dashboard_path = /var/lib/grafana/dashboards/kubeflow.json
```

## 🔧 Advanced Configuration

### 1. **Custom Resource Definitions**
```yaml
# crd-config.yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: custommljobs.kubeflow.org
spec:
  group: kubeflow.org
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              image:
                type: string
              command:
                type: array
                items:
                  type: string
  scope: Namespaced
  names:
    plural: custommljobs
    singular: custommljob
    kind: CustomMLJob
```

### 2. **Network Policies**
```yaml
# network-policies.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: kubeflow-network-policy
  namespace: kubeflow
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: kubeflow
    - namespaceSelector:
        matchLabels:
          name: istio-system
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: kubeflow
    - namespaceSelector:
        matchLabels:
          name: istio-system
```

## ✅ Configuration Validation

### Test Configuration
```bash
# Validate YAML files
kubectl apply --dry-run=client -f config.yaml

# Check configuration
kubectl get configmaps -n kubeflow
kubectl describe configmap kubeflow-config -n kubeflow

# Test component connectivity
kubectl exec -n kubeflow deployment/centraldashboard -- curl -s http://localhost:8080/health
```

### Configuration Checklist
- [ ] All ConfigMaps created successfully
- [ ] RBAC permissions configured
- [ ] Storage classes and PVCs ready
- [ ] Authentication working
- [ ] Network policies applied
- [ ] Monitoring configured
- [ ] Pipeline defaults set

## 🔧 Configuration Best Practices

1. **🔐 Security**: Use RBAC and network policies
2. **📊 Monitoring**: Enable comprehensive monitoring
3. **💾 Storage**: Configure appropriate storage classes
4. **🔄 Backup**: Regular configuration backups
5. **📝 Documentation**: Document all custom configurations
6. **🧪 Testing**: Validate configurations in staging first

---

*Your Kubeflow platform is now configured for production use! 🎯*