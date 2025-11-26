# 🚀 Ultimate Kubernetes Admin Workstation - Helm Chart

**Swiss-army knife for Kubernetes, MongoDB, Ops Manager, networking, storage, and cluster inspection**

A comprehensive Helm chart for deploying a true admin workstation inside your Kubernetes cluster. Pre-loaded with 50+ operational tools for MongoDB operations, TLS debugging, networking diagnostics, storage management, and cluster inspection. Designed for both online and **offline/air-gapped environments**.

Built with **nerdctl + containerd** for native Kubernetes integration.

## 🎯 Features

### Container Runtime
- ✅ **nerdctl + containerd** - Kubernetes-native container runtime
- ✅ **k8s.io namespace** - Direct Kubernetes image integration
- ✅ Automated offline bundle creation with Makefile

### MongoDB Operations
- ✅ **Complete MongoDB client stack**: mongosh, mongodump, mongorestore, bsondump, mongostat, mongotop
- ✅ **X.509 TLS debugging**: openssl, gnutls-bin, certificate verification tools
- ✅ **CA trust integration**: Auto-trust custom CAs mounted at /tls/ca.crt

### Kubernetes Administration
- ✅ **Full K8s tooling**: kubectl v1.31.4, Helm 3, jq, yq, envsubst
- ✅ **Cluster inspection**: Complete pod/service/deployment debugging
- ✅ **Storage management**: tridentctl (NetApp Trident), NFS utilities

### Networking & Diagnostics
- ✅ **Complete network stack**: dig, ping, traceroute, netcat, tcpdump, nmap, curl, wget
- ✅ **Performance testing**: iperf3 for bandwidth analysis
- ✅ **TLS verification**: Certificate chain inspection and validation

### Development Tools
- ✅ **Python 3.12**: With pymongo, kubernetes, pyyaml, requests, jinja2
- ✅ **System debugging**: htop, strace, lsof, iotop, psmisc
- ✅ **File operations**: rsync, git, zip, tar, gzip

### Deployment Flexibility
- ✅ **Offline/Air-gapped deployment support** via `global.imageRegistry`
- ✅ Configurable namespace deployment
- ✅ Flexible service account management (create new or reuse existing)
- ✅ Image pull secrets for private registries
- ✅ Resource limits and requests
- ✅ Security contexts (runAsNonRoot, capabilities)
- ✅ Node selectors, tolerations, and affinity rules
- ✅ Customizable environment variables and volumes
- ✅ Optional health probes

### Compliance & Security
- ✅ **Software Bill of Materials (SBOM)** - Text and JSON formats
- ✅ Complete component inventory with licenses
- ✅ CycloneDX and SPDX-like formats
- ✅ Automated SBOM generation with every build
- ✅ SHA256 checksums in MANIFEST.txt

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- **nerdctl** (for building images) - [Installation Guide](./NERDCTL-GUIDE.md)
- containerd (default Kubernetes runtime)
- For offline deployments: Access to an internal container registry

## 📦 Building the Toolbox Image

This project uses **nerdctl + containerd** instead of Docker:

```bash
# Build the comprehensive toolbox image
make build-image

# Test the image
make test-image

# Create complete offline bundle (image + chart + scripts)
make offline-bundle

# View configuration
make info

# Clean up
make clean
```

For detailed nerdctl usage, see [NERDCTL-GUIDE.md](./NERDCTL-GUIDE.md)

## Quick Start

### Online Deployment (with Internet Access)

```bash
# Install from local chart directory
helm install my-toolbox ./chart \
  -f examples/values-online.yaml \
  -n toolbox --create-namespace

# Access the pod
kubectl -n toolbox get pods
kubectl -n toolbox exec -it deploy/my-toolbox-ultimate-k8s-toolbox -- bash
```

### Offline/Air-gapped Deployment

#### Step 1: Prepare Images (from a machine with internet access)

```bash
# Pull the toolbox image
docker pull ultimate-k8s-toolbox:latest

# Tag for your internal registry
docker tag ultimate-k8s-toolbox:latest myregistry.local:5000/platform/ultimate-k8s-toolbox:latest

# Push to internal registry
docker push myregistry.local:5000/platform/ultimate-k8s-toolbox:latest
```

#### Step 2: Create Image Pull Secret (if needed)

```bash
kubectl create namespace toolbox

kubectl create secret docker-registry regcred \
  --docker-server=myregistry.local:5000 \
  --docker-username=myuser \
  --docker-password=mypass \
  --docker-email=myemail@example.com \
  -n toolbox
```

#### Step 3: Deploy with Offline Values

```bash
# Edit values-offline.yaml to set your registry
# Then install:
helm install my-toolbox ./chart \
  -f examples/values-offline.yaml \
  -n toolbox
```

## Configuration

### Key Values

| Parameter | Description | Default |
|-----------|-------------|---------|
| `global.imageRegistry` | Registry prefix for offline deployments | `""` (Docker Hub) |
| `global.namespaceOverride` | Override deployment namespace | `""` (use release namespace) |
| `image.repository` | Image repository (without registry) | `ultimate-k8s-toolbox` |
| `image.tag` | Image tag | `latest` |
| `imagePullSecrets` | List of image pull secrets | `[]` |
| `serviceAccount.create` | Create service account | `true` |
| `serviceAccount.name` | Service account name (or reuse existing) | `""` |
| `replicaCount` | Number of pod replicas | `1` |
| `resources.requests.cpu` | CPU request | `100m` |
| `resources.requests.memory` | Memory request | `256Mi` |
| `resources.limits.cpu` | CPU limit | `2` |
| `resources.limits.memory` | Memory limit | `4Gi` |

### Image Resolution Logic

The chart uses the `ultimate-k8s-toolbox.image` helper to construct the full image path:

**Online (no registry override):**
```yaml
global.imageRegistry: ""
image.repository: "ultimate-k8s-toolbox"
image.tag: "latest"
# Result: ultimate-k8s-toolbox:latest
```

**Offline with simple registry:**
```yaml
global.imageRegistry: "myregistry.local:5000"
image.repository: "ultimate-k8s-toolbox"
image.tag: "latest"
# Result: myregistry.local:5000/ultimate-k8s-toolbox:latest
```

**Offline with project path:**
```yaml
global.imageRegistry: "harbor.internal.com"
image.repository: "platform/ultimate-k8s-toolbox"
image.tag: "v1.0.0"
# Result: harbor.internal.com/platform/ultimate-k8s-toolbox:v1.0.0
```

## Deployment Scenarios

### Scenario 1: Quick Test (Online)

```bash
helm install test-toolbox ./chart \
  --set image.repository=ubuntu \
  --set image.tag=24.04 \
  -n default
```

### Scenario 2: Production Offline Deployment

```bash
helm install prod-toolbox ./chart \
  -f examples/values-offline.yaml \
  --set global.imageRegistry=harbor.prod.internal \
  --set image.repository=platform/ultimate-k8s-toolbox \
  --set image.tag=v1.0.0 \
  -n toolbox --create-namespace
```

### Scenario 3: MongoDB Namespace with Existing SA

```bash
helm install mongo-toolbox ./chart \
  -f examples/values-mongodb.yaml \
  -n mongodb
```

### Scenario 4: Multiple Environments

```bash
# Development
helm install dev-toolbox ./chart \
  --set global.imageRegistry=registry.dev.local \
  -n dev-tools --create-namespace

# Production
helm install prod-toolbox ./chart \
  --set global.imageRegistry=registry.prod.local \
  --set resources.limits.memory=8Gi \
  -n prod-tools --create-namespace
```

## Accessing the Toolbox

Once deployed, access your toolbox pod:

```bash
# List pods
kubectl -n toolbox get pods

# Execute into the pod
kubectl -n toolbox exec -it deploy/my-toolbox-ultimate-k8s-toolbox -- bash

# Or use the pod name directly
POD_NAME=$(kubectl -n toolbox get pod -l app.kubernetes.io/name=ultimate-k8s-toolbox -o jsonpath='{.items[0].metadata.name}')
kubectl -n toolbox exec -it $POD_NAME -- bash
```

## Offline Deployment Best Practices

### 1. Image Management

Create a script to sync images to your offline registry:

```bash
#!/bin/bash
# sync-images.sh

ONLINE_IMAGE="ultimate-k8s-toolbox:latest"
OFFLINE_REGISTRY="myregistry.local:5000"
OFFLINE_IMAGE="${OFFLINE_REGISTRY}/platform/ultimate-k8s-toolbox:latest"

# Pull from internet
docker pull $ONLINE_IMAGE

# Tag for offline registry
docker tag $ONLINE_IMAGE $OFFLINE_IMAGE

# Push to offline registry
docker push $OFFLINE_IMAGE

echo "Image synced to offline registry: $OFFLINE_IMAGE"
```

### 2. Bundle Chart for Offline Transfer

```bash
# Package the chart
helm package ./ultimate-k8s-toolbox

# This creates: ultimate-k8s-toolbox-0.1.0.tgz
# Transfer this file to your offline environment
```

### 3. Deploy from Package in Offline Environment

```bash
# Install from packaged chart
helm install my-toolbox ultimate-k8s-toolbox-0.1.0.tgz \
  -f values-offline.yaml \
  -n toolbox --create-namespace
```

## Upgrading

```bash
# Upgrade with new values
helm upgrade my-toolbox ./ultimate-k8s-toolbox \
  -f values-offline.yaml \
  -n toolbox

# Upgrade with new image tag
helm upgrade my-toolbox ./ultimate-k8s-toolbox \
  --reuse-values \
  --set image.tag=v1.1.0 \
  -n toolbox
```

## Uninstalling

```bash
helm uninstall my-toolbox -n toolbox
```

## Troubleshooting

### Image Pull Errors in Offline Environment

```bash
# Check image pull secret
kubectl get secret regcred -n toolbox -o yaml

# Verify pod events
kubectl describe pod -n toolbox -l app.kubernetes.io/name=ultimate-k8s-toolbox

# Common issues:
# 1. Wrong registry URL in global.imageRegistry
# 2. Image not pushed to offline registry
# 3. Missing or incorrect imagePullSecrets
# 4. Registry authentication failure
```

### Pod Not Starting

```bash
# Check pod status
kubectl get pods -n toolbox

# View logs
kubectl logs -n toolbox -l app.kubernetes.io/name=ultimate-k8s-toolbox

# Check events
kubectl get events -n toolbox --sort-by='.lastTimestamp'
```

### Testing Registry Connectivity

```bash
# From within the cluster, test registry access
kubectl run test-registry --image=busybox --rm -it --restart=Never -- sh
# Then inside the pod:
wget -O- http://myregistry.local:5000/v2/_catalog
```

## Advanced Configuration

### Custom Environment Variables

```yaml
container:
  env:
    - name: MONGODB_URI
      value: "mongodb://mongodb.mongodb.svc.cluster.local:27017"
    - name: CUSTOM_VAR
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: my-key
```

### Volume Mounts

```yaml
volumes:
  - name: config
    configMap:
      name: toolbox-config
  - name: scripts
    persistentVolumeClaim:
      claimName: scripts-pvc

container:
  volumeMounts:
    - name: config
      mountPath: /config
      readOnly: true
    - name: scripts
      mountPath: /scripts
```

### Health Probes

```yaml
livenessProbe:
  enabled: true
  exec:
    command:
      - /bin/bash
      - -c
      - "ps aux | grep -v grep | grep 'tail -f /dev/null'"
  initialDelaySeconds: 10
  periodSeconds: 30

readinessProbe:
  enabled: true
  exec:
    command:
      - /bin/bash
      - -c
      - "kubectl version --client"
  initialDelaySeconds: 5
  periodSeconds: 10
```

## 📖 Documentation

- **[NERDCTL-GUIDE.md](./NERDCTL-GUIDE.md)** - Complete guide to nerdctl + containerd setup and usage
- **[TOOLS-REFERENCE.md](./TOOLS-REFERENCE.md)** - Comprehensive reference of all 50+ installed tools with examples
- **[QUICKSTART.md](./QUICKSTART.md)** - Get started in 5 minutes
- **[OFFLINE-DEPLOYMENT.md](./OFFLINE-DEPLOYMENT.md)** - Air-gapped deployment guide
- **[MAKEFILE.md](./MAKEFILE.md)** - Makefile targets and automation
- **[INDEX.md](./INDEX.md)** - Project structure and navigation
- **[REORGANIZATION.md](./REORGANIZATION.md)** - Migration guide for new structure

## 🔍 Tool Categories

### MongoDB (Section 1)
mongosh, mongodump, mongorestore, bsondump, mongostat, mongotop, mongofiles, mongoexport, mongoimport

### TLS/X.509 (Section 2)  
openssl, certtool (gnutls-bin), CA trust integration

### Kubernetes (Section 3)
kubectl, helm, jq, yq, envsubst

### Networking (Section 4)
dig, nslookup, ping, traceroute, netcat, tcpdump, nmap, curl, wget, telnet, iperf3

### Storage (Section 5)
tridentctl (NetApp Trident), nfs-common, rsync, git, zip/unzip, tar, gzip

### Python (Section 6)
Python 3.12 + pymongo, kubernetes, pyyaml, requests, jinja2, click

### System Tools (Section 7)
vim, nano, htop, less, psmisc, strace, procps, lsof, iotop, bash-completion

### CA Integration (Section 8)
Auto-trust custom CAs, system-wide certificate management

## 🎯 Quick Tool Reference

```bash
# Inside the toolbox pod, view all tools:
show-versions.sh

# MongoDB with TLS
mongosh "mongodb://host:27017" --tls --tlsCAFile /tls/ca.crt

# Certificate debugging
openssl s_client -connect host:27017 -tls1_2 -CAfile /tls/ca.crt

# DNS troubleshooting
dig +short service.namespace.svc.cluster.local

# Network connectivity
nc -zv service.namespace.svc.cluster.local 27017

# Kubernetes inspection
kubectl get pods -A
helm list -A

# Storage operations
tridentctl get volume
rsync -avz /source/ /dest/

# Python scripting
python3 -c "from pymongo import MongoClient; print('Ready')"
```

## 📋 Software Bill of Materials (SBOM)

Every offline bundle includes comprehensive SBOM documentation:

- **SBOM.txt** - Human-readable format with all components, versions, and licenses
- **SBOM.json** - CycloneDX 1.4 format for automated processing
- **MANIFEST.txt** - SHA256 checksums for verification

The SBOM catalogs all 40 components including:
- Container images and base OS
- MongoDB tools (mongosh, database tools)
- Kubernetes tools (kubectl, Helm, yq)
- Networking utilities (13 tools)
- Python packages (7 packages)
- System tools (10 utilities)

See [SBOM.md](./SBOM.md) for complete documentation.

## License

MIT

## Contributing

Contributions welcome! Please submit pull requests or issues.

---

**Built with nerdctl + containerd for native Kubernetes integration** 🚀
