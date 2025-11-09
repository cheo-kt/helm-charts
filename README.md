# CyberChef Helm Chart Deployment

**Student:** Sergio  
**Student ID:** A00396046  
**Course:** Plataformas 2  
**Date:** November 8, 2025

---

## Project Overview

This project involves deploying CyberChef (a web application for encryption, encoding, compression, and data analysis) using Helm charts on a Kubernetes cluster.

**Repository:** `helm-charts/cyberchef`  
**Chart Version:** 2.0.1  
**App Version:** v9.24.7  
**Image:** `mpepping/cyberchef:v9.24.7`

---

## Initial Analysis

The chart contains the following structure:

```
cyberchef/
├── Chart.yaml
├── README.md
├── values.yaml
└── templates/
    ├── _helpers.tpl
    ├── deployment.yaml
    ├── ingress.yaml
    └── service.yaml
```

### Files Examined:
- **Chart.yaml**: Contains chart metadata and version information
- **values.yaml**: Default configuration values
- **templates/deployment.yaml**: Kubernetes Deployment manifest
- **templates/service.yaml**: Kubernetes Service manifest
- **templates/ingress.yaml**: Kubernetes Ingress manifest (optional)
- **templates/_helpers.tpl**: Helper functions for templating

---

## Issue Identified

During the initial review of the chart templates, a critical port mismatch was discovered:

### The Problem:
1. **In `values.yaml`:**
   ```yaml
   service:
     type: ClusterIP
     port: 8080  # INCORRECT
   ```

2. **In `deployment.yaml`:**
   ```yaml
   ports:
     - name: http
       containerPort: 8000  # Container listens on 8000
   ```

3. **In `ingress.yaml`:**
   ```yaml
   backend:
     service:
       name: {{ $fullName }}
       port:
         number: 8000  # Correctly points to 8000
   ```

### Root Cause:
The CyberChef Docker image (`mpepping/cyberchef:v9.24.7`) runs the application on **port 8000** internally, but the Service was configured to expose **port 8080**. This mismatch would cause connectivity issues when trying to access the application through the Service.

---

## Solution Applied

### Step 1: Fix the Port Configuration

Edit the `values.yaml` file:

```bash
nano values.yaml
```

Change the service port from 8080 to 8000:

```yaml
service:
  type: ClusterIP
  port: 8000  # CORRECTED
  annotations: {}
  labels: {}
```

---

## Deployment Steps

### Prerequisites:
- Kubernetes cluster running (minikube, kind, Docker Desktop, etc.)
- `kubectl` configured and connected to the cluster
- Helm 3.x installed

### Step 2: Install the Helm Chart

Navigate to the parent directory containing the cyberchef folder:

```bash
cd /mnt/d/Backup\ Checho/Octavo\ Semestre/Plataformas\ 2/helm-charts/
```

Install the chart:

```bash
helm install cyberchef ./cyberchef
```

### Step 3: Verify the Deployment

Check the Helm release:

```bash
helm list
```

**Output:**
```
NAME        NAMESPACE   REVISION   UPDATED                                 STATUS     CHART              APP VERSION
cyberchef   default     1          2025-11-08 19:07:56.990432942 -0500    deployed   cyberchef-2.0.1    v9.24.7
```

Check the pods:

```bash
kubectl get pods
```

**Output:**
```
NAME                         READY   STATUS    RESTARTS   AGE
cyberchef-5f65b78dbb-hmwqp   1/1     Running   0          13s
```

Check the services:

```bash
kubectl get svc
```

**Output:**
```
NAME        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
cyberchef   ClusterIP   10.109.98.58   <none>        8000/TCP   21s
```

Verify pod labels:

```bash
kubectl get pods -l app.kubernetes.io/name=cyberchef
```

**Output:**
```
NAME                         READY   STATUS    RESTARTS   AGE
cyberchef-5f65b78dbb-hmwqp   1/1     Running   0          30s
```

---

## Accessing the Application

Since the Service type is `ClusterIP`, the application is only accessible within the cluster. Use port-forwarding to access it locally:

```bash
kubectl port-forward svc/cyberchef 8000:8000
```

Then open your browser and navigate to:
```
http://localhost:8000
```

### Alternative: Expose via NodePort

To access the application without port-forwarding:

```bash
helm upgrade cyberchef ./cyberchef --set service.type=NodePort
```

Get the NodePort assigned:

```bash
kubectl get svc cyberchef
```

Access via: `http://<NODE-IP>:<NODE-PORT>`

---

## Resource Configuration

The current deployment uses default resource settings (no limits or requests defined). For production environments, consider adding:

```bash
helm upgrade cyberchef ./cyberchef \
  --set resources.requests.memory=128Mi \
  --set resources.requests.cpu=100m \
  --set resources.limits.memory=256Mi \
  --set resources.limits.cpu=200m
```

---

## Testing and Validation

### Validate the Chart:

```bash
helm lint ./cyberchef
```

### Dry-run Installation:

```bash
helm install cyberchef ./cyberchef --dry-run --debug
```

### Check Pod Logs:

```bash
kubectl logs -f deployment/cyberchef
```

### Describe Pod for Debugging:

```bash
kubectl describe pod -l app.kubernetes.io/name=cyberchef
```

---

## Cleanup

To remove the deployment:

```bash
helm uninstall cyberchef
```

Verify removal:

```bash
helm list
kubectl get pods
```

---

## Summary

### What Was Done:
1. Analyzed the Helm chart structure and templates
2. Identified a port mismatch between the Service and Deployment
3. Corrected the `values.yaml` file (port 8080 to 8000)
4. Successfully deployed CyberChef using Helm
5. Verified all Kubernetes resources were created correctly
6. Confirmed the application is running and accessible

### Key Learnings:
- Importance of validating port configurations across all Kubernetes resources
- Understanding the relationship between containerPort, Service port, and targetPort
- Using Helm for simplified Kubernetes application deployment
- Troubleshooting and debugging Helm charts before deployment

---
## Photos
<img width="1600" height="183" alt="image" src="https://github.com/user-attachments/assets/63819d1d-a06b-48f6-96bc-025e5918bc75" />
<img width="1600" height="449" alt="image" src="https://github.com/user-attachments/assets/9e72e0ee-cd1d-421e-a43e-54e6da8d6003" />
<img width="1600" height="583" alt="image" src="https://github.com/user-attachments/assets/9a6350d6-ba7d-4ccd-a9aa-9cdebe92d51a" />
<img width="1600" height="907" alt="image" src="https://github.com/user-attachments/assets/d8971a0b-0c5a-45f9-a7a7-8f21f07a02ae" />


---
## References

- [CyberChef GitHub Repository](https://github.com/gchq/CyberChef/)
- [Helm Documentation](https://helm.sh/docs/)
- [Kubernetes Services Documentation](https://kubernetes.io/docs/concepts/services-networking/service/)

---

