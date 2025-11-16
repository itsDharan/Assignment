# Production-Grade WordPress on Kubernetes with Monitoring

This repository contains a complete solution for deploying a production-grade WordPress application on Kubernetes with full monitoring and alerting capabilities using Prometheus and Grafana.

## 📋 Table of Contents

- [Architecture Overview](#architecture-overview)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Project Structure](#project-structure)
- [Objective 1: WordPress Application](#objective-1-wordpress-application)
- [Objective 2: Monitoring and Alerting](#objective-2-monitoring-and-alerting)
- [Testing](#testing)
- [Cleanup](#cleanup)
- [Best Practices Implemented](#best-practices-implemented)

## 🏗️ Architecture Overview

```
Internet
    ↓
[Nginx with OpenResty/Lua]
    ↓ (proxy_pass)
[WordPress Pods]
    ↓
[MySQL StatefulSet]
    ↓
[PersistentVolume with ReadWriteMany]

[Prometheus] → [Grafana]
    ↑
[ServiceMonitor/PodMonitor]
```

## 📦 Prerequisites

- Kubernetes cluster (v1.21+)
- Helm 3.x installed
- kubectl configured
- Docker for building images (optional if using pre-built images)
- Storage class supporting ReadWriteMany (for production scaling)

## 🚀 Quick Start

### Deploy WordPress Application

```bash
# Add required Helm repositories
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Deploy WordPress application
cd wordpress-helm-chart
helm install wordpress . -f values.yaml --create-namespace --namespace wordpress

# Deploy monitoring stack
cd ../monitoring
helm install monitoring . -f values.yaml --create-namespace --namespace monitoring
```

### Access Applications

```bash
# Get WordPress URL
kubectl get svc -n wordpress wordpress-nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}'

# Get Grafana URL
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80

# Default Grafana credentials
# Username: admin
# Password: (retrieve using below command)
kubectl get secret -n monitoring monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```

## 📁 Project Structure

```
.
├── README.md
├── docker/
│   ├── wordpress/
│   │   └── Dockerfile
│   ├── mysql/
│   │   └── Dockerfile
│   └── nginx/
│       ├── Dockerfile
│       └── nginx.conf
├── wordpress-helm-chart/
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│       ├── namespace.yaml
│       ├── pv.yaml
│       ├── pvc.yaml
│       ├── mysql-statefulset.yaml
│       ├── mysql-service.yaml
│       ├── wordpress-deployment.yaml
│       ├── wordpress-service.yaml
│       ├── nginx-configmap.yaml
│       ├── nginx-deployment.yaml
│       ├── nginx-service.yaml
│       └── servicemonitor.yaml
├── monitoring/
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│       ├── prometheus-values.yaml
│       ├── grafana-values.yaml
│       └── alerting-rules.yaml
└── docs/
    ├── metrics-documentation.md
    └── architecture.md
```

## 🎯 Objective 1: WordPress Application

### Features Implemented

- ✅ PersistentVolumeClaims and PersistentVolumes with ReadWriteMany support
- ✅ Custom Dockerfiles for WordPress, MySQL, and Nginx
- ✅ OpenResty with Lua support compiled into Nginx
- ✅ All requests proxy-passed from Nginx to WordPress
- ✅ Helm chart deployment with `helm install` and `helm delete` commands
- ✅ Production-grade configurations with resource limits and health checks

### Key Components

1. **Nginx with OpenResty**: Acts as reverse proxy with Lua scripting capabilities
2. **WordPress**: Scalable deployment with shared storage
3. **MySQL**: StatefulSet for data persistence
4. **Storage**: ReadWriteMany PVC for horizontal scaling

## 🔍 Objective 2: Monitoring and Alerting

### Features Implemented

- ✅ Prometheus deployment for metrics collection
- ✅ Grafana for visualization with pre-configured dashboards
- ✅ Container metrics: CPU utilization, memory usage
- ✅ Nginx metrics: Total request count, 5xx errors
- ✅ Custom alerting rules for critical conditions
- ✅ Kubernetes cluster metrics visualization

### Dashboards Included

1. **Kubernetes Cluster Overview**: Node and pod metrics
2. **WordPress Application Dashboard**: Application-specific metrics
3. **Nginx Performance Dashboard**: Request rates, error rates, latency
4. **MySQL Performance Dashboard**: Query performance, connections

## 🧪 Testing

### Verify WordPress Deployment

```bash
# Check all pods are running
kubectl get pods -n wordpress

# Test WordPress connectivity
curl -I http://$(kubectl get svc -n wordpress wordpress-nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

# Scale WordPress pods
kubectl scale deployment wordpress -n wordpress --replicas=3

# Verify ReadWriteMany volume is working
kubectl exec -n wordpress -it wordpress-<pod-id> -- ls -la /var/www/html
```

### Verify Monitoring

```bash
# Check Prometheus targets
kubectl port-forward -n monitoring svc/prometheus-server 9090:80
# Visit http://localhost:9090/targets

# Check metrics are being collected
kubectl exec -n monitoring prometheus-server-0 -- promtool query instant http://localhost:9090 'up{job="wordpress"}'
```

## 🧹 Cleanup

```bash
# Remove WordPress application
helm delete wordpress -n wordpress
kubectl delete namespace wordpress

# Remove monitoring stack
helm delete monitoring -n monitoring
kubectl delete namespace monitoring

# Clean up PVs (if using local storage)
kubectl delete pv --all
```

## 💡 Best Practices Implemented

### Security
- ✅ Non-root containers
- ✅ Network policies for pod communication
- ✅ Secret management for passwords
- ✅ Resource quotas and limits

### Reliability
- ✅ Health checks and readiness probes
- ✅ Graceful shutdown handling
- ✅ Persistent storage for stateful components
- ✅ Anti-affinity rules for HA

### Observability
- ✅ Structured logging
- ✅ Comprehensive metrics
- ✅ Distributed tracing ready
- ✅ Alert rules for common issues

### Performance
- ✅ Horizontal Pod Autoscaling configured
- ✅ Resource requests and limits
- ✅ Connection pooling for database
- ✅ Nginx caching enabled

## 📊 Metrics Documentation

For detailed metrics documentation, see [docs/metrics-documentation.md](docs/metrics-documentation.md)

## 🤝 Contributing

This project follows GitFlow branching strategy. Please create feature branches from `develop` and submit PRs.

## 📝 License

MIT License - See LICENSE file for details

## 👨‍💻 Author

Infrastructure Team - Syfe 2022 Internship Submission

---

**Note**: This is a production-grade implementation suitable for real-world deployments. All configurations follow Kubernetes and cloud-native best practices.
