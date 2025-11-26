# Offline Deployment Guide for Ultimate K8s Toolbox

This guide walks through the complete process of deploying the ultimate-k8s-toolbox in an **offline/air-gapped** Kubernetes environment.

## Prerequisites

- Source environment with internet access (for image preparation)
- Target offline Kubernetes cluster
- Internal/private container registry accessible from the offline cluster
- Helm 3.0+ installed on both environments
- kubectl configured for both environments

---

## Part 1: Preparation (Internet-Connected Environment)

### Step 1: Build or Pull the Toolbox Image

If you have a pre-built image:
```bash
# Pull from Docker Hub or your registry
docker pull ultimate-k8s-toolbox:latest
```

Or build from Dockerfile:
```bash
# Build the image
docker build -t ultimate-k8s-toolbox:latest .

# Tag with a version
docker tag ultimate-k8s-toolbox:latest ultimate-k8s-toolbox:v1.0.0
```

### Step 2: Export the Image

```bash
# Save the image to a tar file
docker save ultimate-k8s-toolbox:v1.0.0 -o ultimate-k8s-toolbox-v1.0.0.tar

# Verify the tar file
ls -lh ultimate-k8s-toolbox-v1.0.0.tar
```

### Step 3: Package the Helm Chart

```bash
# Package the chart
cd /path/to/charts
helm package chart/

# This creates: ultimate-k8s-toolbox-0.1.0.tgz
```

### Step 4: Create Transfer Bundle

```bash
# Create a directory for all offline artifacts
mkdir -p offline-bundle
cp ultimate-k8s-toolbox-v1.0.0.tar offline-bundle/
cp ultimate-k8s-toolbox-0.1.0.tgz offline-bundle/
cp ultimate-k8s-toolbox/values-offline.yaml offline-bundle/

# Optional: Create a manifest file
cat > offline-bundle/manifest.txt <<EOF
Ultimate K8s Toolbox - Offline Deployment Bundle
Version: 1.0.0
Chart Version: 0.1.0
Created: $(date)

Contents:
- ultimate-k8s-toolbox-v1.0.0.tar (Docker image)
- ultimate-k8s-toolbox-0.1.0.tgz (Helm chart)
- values-offline.yaml (Configuration template)
EOF

# Create a compressed archive
tar -czf ultimate-k8s-toolbox-offline-bundle.tar.gz offline-bundle/

echo "Offline bundle created: ultimate-k8s-toolbox-offline-bundle.tar.gz"
ls -lh ultimate-k8s-toolbox-offline-bundle.tar.gz
```

---

## Part 2: Transfer to Offline Environment

Transfer `ultimate-k8s-toolbox-offline-bundle.tar.gz` to your offline environment using approved methods:
- USB drive
- Secure file transfer
- Approved network transfer mechanism

---

## Part 3: Deployment (Offline Environment)

### Step 1: Extract the Bundle

```bash
# Extract the bundle
tar -xzf ultimate-k8s-toolbox-offline-bundle.tar.gz
cd offline-bundle/

# Verify contents
ls -la
```

### Step 2: Load Image to Internal Registry

#### Option A: Direct Push (if you have docker/podman on the offline system)

```bash
# Load the image
docker load -i ultimate-k8s-toolbox-v1.0.0.tar

# Verify image loaded
docker images | grep ultimate-k8s-toolbox

# Tag for your internal registry
# Replace with your actual registry URL
INTERNAL_REGISTRY="harbor.internal.company.com"
PROJECT="platform"

docker tag ultimate-k8s-toolbox:v1.0.0 \
  ${INTERNAL_REGISTRY}/${PROJECT}/ultimate-k8s-toolbox:v1.0.0

docker tag ultimate-k8s-toolbox:v1.0.0 \
  ${INTERNAL_REGISTRY}/${PROJECT}/ultimate-k8s-toolbox:latest

# Login to internal registry
docker login ${INTERNAL_REGISTRY}

# Push to internal registry
docker push ${INTERNAL_REGISTRY}/${PROJECT}/ultimate-k8s-toolbox:v1.0.0
docker push ${INTERNAL_REGISTRY}/${PROJECT}/ultimate-k8s-toolbox:latest
```

#### Option B: Using Skopeo (alternative tool)

```bash
# Load and push in one command using skopeo
skopeo copy \
  docker-archive:ultimate-k8s-toolbox-v1.0.0.tar \
  docker://harbor.internal.company.com/platform/ultimate-k8s-toolbox:v1.0.0
```

### Step 3: Create Registry Credentials Secret

If your internal registry requires authentication:

```bash
# Create namespace
kubectl create namespace toolbox

# Create image pull secret
kubectl create secret docker-registry regcred \
  --docker-server=harbor.internal.company.com \
  --docker-username=YOUR_USERNAME \
  --docker-password=YOUR_PASSWORD \
  --docker-email=YOUR_EMAIL \
  -n toolbox

# Verify secret
kubectl get secret regcred -n toolbox
```

### Step 4: Configure Values File

Edit `values-offline.yaml` with your environment details:

```yaml
global:
  # Your internal registry URL
  imageRegistry: "harbor.internal.company.com"
  namespaceOverride: ""

image:
  # Image path within the registry
  repository: "platform/ultimate-k8s-toolbox"
  tag: "v1.0.0"
  pullPolicy: IfNotPresent

# Reference your registry secret
imagePullSecrets:
  - name: regcred

serviceAccount:
  create: true
  name: ""

resources:
  requests:
    cpu: "100m"
    memory: "256Mi"
  limits:
    cpu: "2"
    memory: "4Gi"
```

### Step 5: Deploy with Helm

```bash
# Install the chart
helm install my-toolbox ultimate-k8s-toolbox-0.1.0.tgz \
  -f values-offline.yaml \
  -n toolbox

# Monitor deployment
kubectl get pods -n toolbox -w
```

### Step 6: Verify Deployment

```bash
# Check deployment status
kubectl get all -n toolbox

# Get pod name
POD_NAME=$(kubectl get pod -n toolbox -l app.kubernetes.io/name=ultimate-k8s-toolbox -o jsonpath='{.items[0].metadata.name}')

# Check pod details
kubectl describe pod $POD_NAME -n toolbox

# View logs
kubectl logs $POD_NAME -n toolbox

# Test access
kubectl exec -n toolbox $POD_NAME -- bash -c "echo 'Pod is accessible!'"
```

---

## Part 4: Multiple Namespace Deployment

Deploy to multiple namespaces with different configurations:

### Development Environment
```bash
helm install dev-toolbox ultimate-k8s-toolbox-0.1.0.tgz \
  --set global.imageRegistry="harbor.internal.company.com" \
  --set image.repository="platform/ultimate-k8s-toolbox" \
  --set image.tag="v1.0.0" \
  --set imagePullSecrets[0].name="regcred" \
  -n dev-tools --create-namespace
```

### Production Environment
```bash
helm install prod-toolbox ultimate-k8s-toolbox-0.1.0.tgz \
  --set global.imageRegistry="harbor.internal.company.com" \
  --set image.repository="platform/ultimate-k8s-toolbox" \
  --set image.tag="v1.0.0" \
  --set imagePullSecrets[0].name="regcred" \
  --set resources.limits.memory="8Gi" \
  --set resources.limits.cpu="4" \
  -n prod-tools --create-namespace
```

### MongoDB Namespace (Reusing Existing SA)
```bash
helm install mongo-toolbox ultimate-k8s-toolbox-0.1.0.tgz \
  -f values-mongodb.yaml \
  --set global.imageRegistry="harbor.internal.company.com" \
  -n mongodb
```

---

## Part 5: Accessing the Toolbox

Once deployed, access your toolbox:

```bash
# Get pod name
POD_NAME=$(kubectl get pod -n toolbox -l app.kubernetes.io/name=ultimate-k8s-toolbox -o jsonpath='{.items[0].metadata.name}')

# Execute into the pod
kubectl exec -n toolbox -it $POD_NAME -- bash

# Or using deployment name
kubectl exec -n toolbox -it deploy/my-toolbox-ultimate-k8s-toolbox -- bash
```

Inside the pod, you'll have access to:
- MongoDB tools (mongosh)
- Kubernetes CLI (kubectl)
- Trident CLI (tridentctl)
- Network utilities (ping, curl, wget, netcat, nslookup, dig, traceroute)
- And more...

---

## Part 6: Upgrading

### Prepare New Version

In the online environment:
```bash
# Build/pull new version
docker pull ultimate-k8s-toolbox:v1.1.0

# Export and transfer
docker save ultimate-k8s-toolbox:v1.1.0 -o ultimate-k8s-toolbox-v1.1.0.tar
```

### Deploy Upgrade in Offline Environment

```bash
# Load new image
docker load -i ultimate-k8s-toolbox-v1.1.0.tar

# Tag and push to internal registry
docker tag ultimate-k8s-toolbox:v1.1.0 \
  harbor.internal.company.com/platform/ultimate-k8s-toolbox:v1.1.0
docker push harbor.internal.company.com/platform/ultimate-k8s-toolbox:v1.1.0

# Upgrade Helm release
helm upgrade my-toolbox ultimate-k8s-toolbox-0.1.0.tgz \
  --reuse-values \
  --set image.tag=v1.1.0 \
  -n toolbox
```

---

## Part 7: Troubleshooting

### Image Pull Errors

```bash
# Check if image exists in registry
curl -u username:password https://harbor.internal.company.com/v2/platform/ultimate-k8s-toolbox/tags/list

# Verify secret
kubectl get secret regcred -n toolbox -o yaml

# Check pod events
kubectl describe pod -n toolbox -l app.kubernetes.io/name=ultimate-k8s-toolbox
```

### Pod Not Starting

```bash
# Check pod status
kubectl get pods -n toolbox

# View detailed events
kubectl get events -n toolbox --sort-by='.lastTimestamp'

# Check logs
kubectl logs -n toolbox -l app.kubernetes.io/name=ultimate-k8s-toolbox

# Describe deployment
kubectl describe deployment -n toolbox
```

### Registry Connectivity

```bash
# Test registry from within cluster
kubectl run test-registry --image=busybox --rm -it --restart=Never -- sh

# Inside the test pod:
wget -O- http://harbor.internal.company.com/v2/_catalog
# or
nslookup harbor.internal.company.com
```

### Verify Image in Registry

```bash
# Using curl
curl -k -u username:password \
  https://harbor.internal.company.com/v2/platform/ultimate-k8s-toolbox/manifests/v1.0.0

# Using skopeo
skopeo inspect docker://harbor.internal.company.com/platform/ultimate-k8s-toolbox:v1.0.0
```

---

## Part 8: Cleanup

```bash
# Uninstall release
helm uninstall my-toolbox -n toolbox

# Delete namespace (if desired)
kubectl delete namespace toolbox

# Remove image from internal registry (optional)
# This depends on your registry's API/UI
```

---

## Quick Reference Commands

```bash
# List all Helm releases
helm list -A

# Get release values
helm get values my-toolbox -n toolbox

# Check Helm release history
helm history my-toolbox -n toolbox

# Rollback to previous version
helm rollback my-toolbox 1 -n toolbox

# Package chart with specific version
helm package chart/ --version 0.2.0

# Export all Helm values
helm get values my-toolbox -n toolbox -o yaml > current-values.yaml
```

---

## Security Considerations

1. **Image Scanning**: Scan images before importing to offline environment
2. **Registry Security**: Use TLS and authentication for internal registry
3. **Secret Management**: Rotate registry credentials regularly
4. **RBAC**: Review and minimize service account permissions
5. **Network Policies**: Implement network policies if required
6. **Pod Security**: Enable pod security standards/policies

---

## Registry-Specific Guides

### Harbor Registry

```bash
# Create project
harbor create-project platform

# Push image
docker login harbor.internal.company.com
docker push harbor.internal.company.com/platform/ultimate-k8s-toolbox:v1.0.0
```

### Artifactory

```bash
# Create Docker repository
# Push using Docker
docker login artifactory.internal.company.com
docker push artifactory.internal.company.com/docker-local/ultimate-k8s-toolbox:v1.0.0
```

### Nexus Registry

```bash
# Docker hosted repository
docker login nexus.internal.company.com:8082
docker push nexus.internal.company.com:8082/ultimate-k8s-toolbox:v1.0.0
```

---

## Automation Script

Create a script to automate offline bundle creation:

```bash
#!/bin/bash
# create-offline-bundle.sh

VERSION=${1:-v1.0.0}
CHART_VERSION=${2:-0.1.0}

echo "Creating offline bundle for version $VERSION..."

# Build/pull image
docker pull ultimate-k8s-toolbox:latest
docker tag ultimate-k8s-toolbox:latest ultimate-k8s-toolbox:$VERSION

# Export image
docker save ultimate-k8s-toolbox:$VERSION -o ultimate-k8s-toolbox-${VERSION}.tar

# Package chart
helm package chart/

# Create bundle
mkdir -p offline-bundle
mv ultimate-k8s-toolbox-${VERSION}.tar offline-bundle/
mv ultimate-k8s-toolbox-${CHART_VERSION}.tgz offline-bundle/
cp values-offline.yaml offline-bundle/

# Create archive
tar -czf ultimate-k8s-toolbox-offline-${VERSION}.tar.gz offline-bundle/

echo "Bundle created: ultimate-k8s-toolbox-offline-${VERSION}.tar.gz"
ls -lh ultimate-k8s-toolbox-offline-${VERSION}.tar.gz
```

Usage:
```bash
chmod +x create-offline-bundle.sh
./create-offline-bundle.sh v1.0.0 0.1.0
```

---

This guide provides a complete workflow for deploying the ultimate-k8s-toolbox in restricted, air-gapped environments while maintaining flexibility and security.
