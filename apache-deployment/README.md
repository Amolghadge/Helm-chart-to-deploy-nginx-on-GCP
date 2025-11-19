# Apache Deployment to GKE

This directory contains Kubernetes manifests for deploying Apache web server to Google Kubernetes Engine (GKE) without using Helm charts.

## Directory Structure

```
apache-deployment/
├── configmap.yaml      # Apache configuration and HTML content
├── deployment.yaml     # Apache deployment with 3 replicas
└── service.yaml        # LoadBalancer service to expose Apache
```

## Files Overview

### 1. configmap.yaml
Contains:
- **httpd.conf**: Apache HTTP server configuration
- **index.html**: Welcome page with styling

### 2. deployment.yaml
Features:
- **2 replicas** with rolling update strategy
- **Latest Apache image** from Docker Hub (httpd:latest)
- **Resource limits**: 100m-500m CPU, 128Mi-512Mi memory
- **Health checks**: Liveness and readiness probes
- **Pod anti-affinity**: Spreads pods across different nodes
- **Graceful shutdown**: 15s pre-stop delay with 30s termination grace period
- **Volume mounts**: Uses ConfigMap for configuration and HTML

### 3. service.yaml
Provides:
- **LoadBalancer** service type to expose Apache externally
- **Port 80** mapping
- **Session affinity** for client IP persistence

## Prerequisites

1. **GKE Cluster**: Must be running with proper credentials
2. **GitHub Secrets**: Configure in your repository
3. **IAM Permissions**: Service account must have necessary GKE access

## GitHub Secrets Configuration

Add the following secret to your GitHub repository (Settings → Secrets and Variables → Actions):

- **GCP_SA_KEY**: Your GCP service account JSON key with GKE permissions

## Deployment Methods

### Option 1: Automatic (GitHub Actions)
Push to `main` or `develop` branch to trigger automatic deployment:
```bash
git push origin main
```

### Option 2: Manual (kubectl)
```bash
# Apply ConfigMap
kubectl apply -f apache-deployment/configmap.yaml

# Apply Deployment
kubectl apply -f apache-deployment/deployment.yaml

# Apply Service
kubectl apply -f apache-deployment/service.yaml
```

### Option 3: Manual (GitHub Actions)
Go to Actions → Deploy Apache to GKE → Run workflow

## Accessing Apache

### Via LoadBalancer External IP
```bash
# Get external IP
kubectl get svc apache-service -n default

# Access via browser
http://<EXTERNAL-IP>
```

### Via Port Forward (Local Development)
```bash
kubectl port-forward -n default svc/apache-service 8080:80
# Then visit: http://localhost:8080
```

## Monitoring and Verification

### Check Deployment Status
```bash
kubectl get deployment apache-deployment -n default
kubectl get pods -l app=apache -n default
kubectl get svc apache-service -n default
```

### View Pod Logs
```bash
kubectl logs -l app=apache -n default
```

### Describe Resources
```bash
kubectl describe deployment apache-deployment -n default
kubectl describe svc apache-service -n default
```

### Watch Rollout Status
```bash
kubectl rollout status deployment/apache-deployment -n default
```

## Updating Apache

### Update Apache Configuration
1. Edit `configmap.yaml`
2. Apply the changes: `kubectl apply -f apache-deployment/configmap.yaml`
3. Restart pods: `kubectl rollout restart deployment/apache-deployment -n default`

### Update Apache Image
Edit `deployment.yaml` and change the `image` field, then apply:
```bash
kubectl apply -f apache-deployment/deployment.yaml
```

## Scaling

### Scale Replicas
```bash
kubectl scale deployment apache-deployment -n default --replicas=5
```

Or edit `deployment.yaml` and change `spec.replicas` value.

## Deleting Deployment

```bash
kubectl delete -f apache-deployment/
```

Or individual resources:
```bash
kubectl delete deployment apache-deployment -n default
kubectl delete svc apache-service -n default
kubectl delete configmap apache-config -n default
```

## Troubleshooting

### Pods not starting
```bash
kubectl describe pod <pod-name> -n default
kubectl logs <pod-name> -n default
```

### Service not getting external IP
```bash
kubectl describe svc apache-service -n default
# Check Events section for issues
```

### Connection issues
```bash
kubectl port-forward svc/apache-service 8080:80 -n default
# Check firewall and network policies
```

## GKE Cluster Details

- **Project ID**: ornate-producer-477604-s3
- **Cluster Name**: gke-standard-cluster
- **Location**: us-central1-a
- **Namespace**: default

## GitHub Actions Workflow

The workflow file `deploy-apache.yaml` includes:

1. **Checkout code**
2. **Google Cloud authentication**
3. **GKE credentials setup**
4. **kubectl installation**
5. **Namespace creation**
6. **ConfigMap application**
7. **Deployment application**
8. **Service application**
9. **Status verification**
10. **Verification job** (confirms all pods are running)

## Notes

- Apache is deployed with rolling updates enabled
- Each pod has resource requests and limits for proper scheduling
- Liveness and readiness probes ensure healthy containers
- LoadBalancer service may take a few minutes to get an external IP
- Pre-stop lifecycle hook allows graceful connection drainage

