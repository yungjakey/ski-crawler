# Kubernetes Implementation Guide

## Prerequisites
- Access to existing Kubernetes cluster
- kubectl configured
- Helm installed
- GitHub repository access
- AWS S3 access for screenshot storage

## Implementation Steps

### 1. Namespace Setup
```bash
# Create namespace
kubectl create namespace ski-crawler

# Set context
kubectl config set-context --current --namespace=ski-crawler
```

### 2. Database Setup

Create PostgreSQL StatefulSet configuration:

```yaml
# postgres-statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
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
        image: postgres:11.22
        env:
        - name: POSTGRES_DB
          value: ski_crawler
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: username
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
  volumeClaimTemplates:
  - metadata:
      name: postgres-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```

Apply the configuration:
```bash
kubectl apply -f postgres-statefulset.yaml
```

### 3. Application Deployment

Create deployment configuration:

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ski-crawler
spec:
  replicas: 2
  selector:
    matchLabels:
      app: ski-crawler
  template:
    metadata:
      labels:
        app: ski-crawler
    spec:
      containers:
      - name: ski-crawler
        image: <your-registry>/ski-crawler:latest
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        env:
        - name: DB_HOST
          value: postgres
        - name: DB_PORT
          value: "5432"
        - name: DB_NAME
          value: ski_crawler
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        - name: AWS_ACCESS_KEY_ID
          valueFrom:
            secretKeyRef:
              name: aws-secret
              key: access-key
        - name: AWS_SECRET_ACCESS_KEY
          valueFrom:
            secretKeyRef:
              name: aws-secret
              key: secret-key
        - name: S3_BUCKET
          value: ski-crawler-screenshots
```

### 4. Service and Ingress Setup

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: ski-crawler
spec:
  selector:
    app: ski-crawler
  ports:
  - port: 80
    targetPort: 3000
  type: ClusterIP

---
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ski-crawler
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  rules:
  - host: ski-crawler.your-domain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: ski-crawler
            port:
              number: 80
  tls:
  - hosts:
    - ski-crawler.your-domain.com
    secretName: ski-crawler-tls
```

### 5. Secrets Management

```yaml
# secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: postgres-secret
type: Opaque
data:
  username: <base64-encoded-username>
  password: <base64-encoded-password>
---
apiVersion: v1
kind: Secret
metadata:
  name: aws-secret
type: Opaque
data:
  access-key: <base64-encoded-access-key>
  secret-key: <base64-encoded-secret-key>
```

### 6. CI/CD Pipeline

```yaml
# .github/workflows/k8s-deploy.yml
name: Deploy to Kubernetes
on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2

    - name: Build and push Docker image
      uses: docker/build-push-action@v2
      with:
        context: .
        push: true
        tags: <your-registry>/ski-crawler:${{ github.sha }}

    - name: Deploy to Kubernetes
      uses: steebchen/kubectl@v2
      with:
        config: ${{ secrets.KUBE_CONFIG_DATA }}
        command: set image deployment/ski-crawler ski-crawler=<your-registry>/ski-crawler:${{ github.sha }} -n ski-crawler
```

## Monitoring Setup

### 1. Prometheus Configuration

```yaml
# prometheus-servicemonitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: ski-crawler
spec:
  selector:
    matchLabels:
      app: ski-crawler
  endpoints:
  - port: metrics
```

### 2. Grafana Dashboard

Import the following dashboard configuration:
```json
{
  "dashboard": {
    "title": "Ski Crawler Metrics",
    "panels": [
      {
        "title": "CPU Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "container_cpu_usage_seconds_total{namespace='ski-crawler'}"
          }
        ]
      },
      {
        "title": "Memory Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "container_memory_usage_bytes{namespace='ski-crawler'}"
          }
        ]
      }
    ]
  }
}
```

## Resource Management

### 1. Resource Quotas
```yaml
# quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: ski-crawler-quota
spec:
  hard:
    requests.cpu: "2"
    requests.memory: 4Gi
    limits.cpu: "4"
    limits.memory: 8Gi
```

### 2. HPA Configuration
```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: ski-crawler
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ski-crawler
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80
```

## Cleanup Procedure

```bash
# Delete all resources in namespace
kubectl delete namespace ski-crawler

# Remove ingress configuration
kubectl delete ingress ski-crawler

# Remove TLS secrets
kubectl delete secret ski-crawler-tls
```

## Troubleshooting Guide

### Common Issues

1. Pod Status Check
```bash
kubectl get pods -n ski-crawler
kubectl describe pod <pod-name> -n ski-crawler
```

2. Database Connection
```bash
kubectl exec -it <postgres-pod> -- psql -U <username> -d ski_crawler
```

3. Logs Access
```bash
kubectl logs -f deployment/ski-crawler
kubectl logs -f statefulset/postgres
```

### Support
