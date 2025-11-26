# 🚀 Ultimate K8s Admin Workstation - Index

**Swiss-army knife for Kubernetes, MongoDB, Ops Manager, networking, storage, and cluster inspection**

Built with **nerdctl + containerd** | 50+ tools pre-installed | True admin workstation inside your cluster

## Quick Navigation

### 📖 Documentation
1. **[README.md](README.md)** - Main documentation with complete feature overview
2. **[NERDCTL-GUIDE.md](NERDCTL-GUIDE.md)** - Complete nerdctl + containerd setup and usage guide
3. **[TOOLS-REFERENCE.md](TOOLS-REFERENCE.md)** - Comprehensive reference of all 50+ tools with examples
4. **[QUICKSTART.md](QUICKSTART.md)** - Fast deployment guide for immediate use
5. **[OFFLINE-DEPLOYMENT.md](OFFLINE-DEPLOYMENT.md)** - Complete air-gapped deployment guide
6. **[MAKEFILE.md](MAKEFILE.md)** - Offline bundle creation guide
7. **[SBOM.md](SBOM.md)** - Software Bill of Materials documentation
8. **[REORGANIZATION.md](REORGANIZATION.md)** - Project structure migration guide

### 🔧 Configuration Files
- **[chart/values.yaml](chart/values.yaml)** - Default configuration with all options
- **[examples/values-online.yaml](examples/values-online.yaml)** - Internet-connected deployment
- **[examples/values-offline.yaml](examples/values-offline.yaml)** - Air-gapped deployment template  
- **[examples/values-mongodb.yaml](examples/values-mongodb.yaml)** - MongoDB namespace example

### 📋 Chart Files
- **[chart/Chart.yaml](chart/Chart.yaml)** - Chart metadata
- **[chart/.helmignore](chart/.helmignore)** - Helm packaging exclusions
- **[chart/templates/_helpers.tpl](chart/templates/_helpers.tpl)** - Helper templates
- **[chart/templates/serviceaccount.yaml](chart/templates/serviceaccount.yaml)** - ServiceAccount
- **[chart/templates/deployment.yaml](chart/templates/deployment.yaml)** - Deployment

### 🛠️ Build & Deploy Tools
- **[Makefile](Makefile)** - Automated offline bundle creation
- **[build/Dockerfile](build/Dockerfile)** - Toolbox image with all tools
- **[scripts/deploy-offline.sh.template](scripts/deploy-offline.sh.template)** - Offline deployment automation

### 🧪 Tests & Examples
- **[tests/test-helm-chart.sh](tests/test-helm-chart.sh)** - Automated testing script
- **[tests/TEST-RESULTS.md](tests/TEST-RESULTS.md)** - Validation and test results
- **[examples/DEPLOYMENT-EXAMPLES.sh](examples/DEPLOYMENT-EXAMPLES.sh)** - 10 deployment scenarios

---

## Quick Start (30 seconds)

```bash
# Online deployment
helm install my-toolbox chart/ --set image.repository=ubuntu --set image.tag=24.04 -n toolbox --create-namespace

# Offline deployment
helm install my-toolbox chart/ -f examples/values-offline.yaml -n toolbox --create-namespace
```

---

## Features at a Glance

### 🎯 Container Runtime
✅ **nerdctl + containerd** - Kubernetes-native container runtime
✅ **k8s.io namespace** - Direct Kubernetes image integration  
✅ **Automated builds** - Makefile targets for image building and offline bundles

### 📦 Pre-installed Tools (50+)

**MongoDB Client Stack (Section 1)**
- mongosh, mongodump, mongorestore, bsondump, mongostat, mongotop, mongofiles, mongoexport, mongoimport

**X.509 / TLS Tools (Section 2)**
- openssl, certtool (gnutls-bin), CA trust integration

**Kubernetes Admin (Section 3)**
- kubectl v1.31.4, Helm 3, jq, yq, envsubst

**Networking Tools (Section 4)**
- dig, nslookup, ping, traceroute, netcat, tcpdump, nmap, curl, wget, telnet, iperf3

**Storage Tools (Section 5)**
- tridentctl (NetApp Trident), nfs-common, rsync, git, zip/unzip, tar, gzip

**Python Environment (Section 6)**
- Python 3.12 + pymongo, kubernetes, pyyaml, requests, jinja2, click

**System Tools (Section 7)**
- vim, nano, htop, less, psmisc, strace, procps, lsof, iotop, bash-completion

**CA Integration (Section 8)**
- Auto-trust custom CAs mounted at /tls/ca.crt

### 🚀 Deployment Features
✅ **Offline/Air-gapped Support**
- Configurable image registry prefix (`global.imageRegistry`)
- Image pull secrets support
- Works with Harbor, Nexus, Artifactory, and any Docker registry

✅ **Flexible Configuration**
- Namespace override capability
- Service account creation or reuse
- Resource limits and requests
- Security contexts
- Node selectors and affinity rules

✅ **Production Ready**
- Helm best practices
- Secure defaults
- Comprehensive documentation
- Tested and validated

---

## Deployment Scenarios

| Scenario | Values File | Documentation |
|----------|-------------|---------------|
| Online (Internet) | `values-online.yaml` | [QUICKSTART.md](QUICKSTART.md) |
| Offline (Air-gap) | `values-offline.yaml` | [OFFLINE-DEPLOYMENT.md](OFFLINE-DEPLOYMENT.md) |
| MongoDB namespace | `values-mongodb.yaml` | [README.md](README.md) |
| Custom deployment | Create your own | [README.md](README.md) |

---

## File Overview

### Documentation Files (1,493 lines)
- `README.md` - 356 lines - Main documentation
- `OFFLINE-DEPLOYMENT.md` - 507 lines - Offline deployment guide
- `QUICKSTART.md` - 185 lines - Quick reference
- `TEST-RESULTS.md` - 245 lines - Test validation
- `DEPLOYMENT-EXAMPLES.sh` - 289 lines - Example commands

### Configuration Files (323 lines)
- `values.yaml` - 192 lines - Default configuration
- `values-online.yaml` - 24 lines - Online deployment
- `values-offline.yaml` - 57 lines - Offline deployment
- `values-mongodb.yaml` - 36 lines - MongoDB example
- `Chart.yaml` - 15 lines - Chart metadata

### Template Files (218 lines)
- `templates/_helpers.tpl` - 108 lines - Helper functions
- `templates/deployment.yaml` - 97 lines - Deployment template
- `templates/serviceaccount.yaml` - 13 lines - ServiceAccount template

### Tools (235 lines)
- `test-helm-chart.sh` - 235 lines - Automated testing

**Total: 2,389 lines across 15 files**

---

## Common Commands

### Building with nerdctl

```bash
# Build the comprehensive toolbox image
make build-image

# Test the built image
make test-image

# Create complete offline bundle
make offline-bundle

# View configuration
make info

# Clean up
make clean
```

### Deploying with Helm

```bash
# Install
helm install my-toolbox . -n toolbox --create-namespace

# Upgrade
helm upgrade my-toolbox . --reuse-values -n toolbox

## Getting Help

1. **Quick questions?** → [QUICKSTART.md](QUICKSTART.md)
2. **nerdctl setup?** → [NERDCTL-GUIDE.md](NERDCTL-GUIDE.md)
3. **Tool reference?** → [TOOLS-REFERENCE.md](TOOLS-REFERENCE.md)
4. **Offline deployment?** → [OFFLINE-DEPLOYMENT.md](OFFLINE-DEPLOYMENT.md)
5. **Detailed info?** → [README.md](README.md)
6. **Test results?** → [tests/TEST-RESULTS.md](tests/TEST-RESULTS.md)
7. **Examples?** → [examples/DEPLOYMENT-EXAMPLES.sh](examples/DEPLOYMENT-EXAMPLES.sh)
8. **Build automation?** → [MAKEFILE.md](MAKEFILE.md)
kubectl logs -n toolbox -l app.kubernetes.io/name=ultimate-k8s-toolbox

# Package
helm package .

# Lint
helm lint .

# Template
helm template my-toolbox . -f values-offline.yaml
```

### Using Tools Inside Pod

```bash
# View all installed tools
show-versions.sh

# MongoDB with TLS
mongosh "mongodb://host:27017" --tls --tlsCAFile /tls/ca.crt

# Certificate debugging
openssl s_client -connect host:27017 -tls1_2 -CAfile /tls/ca.crt

# Network troubleshooting
dig +short service.namespace.svc.cluster.local
nc -zv service.namespace.svc.cluster.local 27017

# Kubernetes operations
kubectl get pods -A
helm list -A

# Storage operations
tridentctl get volume
rsync -avz /source/ /dest/
```

---

## Getting Help

1. **Quick questions?** → [QUICKSTART.md](QUICKSTART.md)
2. **Offline deployment?** → [OFFLINE-DEPLOYMENT.md](OFFLINE-DEPLOYMENT.md)
3. **Detailed info?** → [README.md](README.md)
4. **Test results?** → [TEST-RESULTS.md](TEST-RESULTS.md)
5. **Examples?** → [DEPLOYMENT-EXAMPLES.sh](DEPLOYMENT-EXAMPLES.sh)

---

## Support Matrix

| Feature | Supported |
|---------|-----------|
| Kubernetes 1.19+ | ✅ |
| Helm 3.0+ | ✅ |
| Online deployment | ✅ |
| Offline deployment | ✅ |
| Multiple registries | ✅ |
| Multiple namespaces | ✅ |
| Service account reuse | ✅ |
| Image pull secrets | ✅ |
| Resource limits | ✅ |
| Security contexts | ✅ |
| Health probes | ✅ |
| Volume mounts | ✅ |

---

ultimate-k8s-toolbox/
├── Chart.yaml                       # Chart metadata
├── values.yaml                      # Default configuration
├── .helmignore                      # Package exclusions
│
├── 📖 Documentation
│   ├── INDEX.md                     # This navigation file
│   ├── README.md                    # Main documentation
│   ├── NERDCTL-GUIDE.md            # nerdctl + containerd setup guide
│   ├── TOOLS-REFERENCE.md          # Complete reference for all 50+ tools
│   ├── QUICKSTART.md               # Quick reference
│   ├── OFFLINE-DEPLOYMENT.md       # Offline deployment guide
│   ├── MAKEFILE.md                 # Build automation guide
│   └── REORGANIZATION.md           # Project structure migration guide
│
├── 🛠️ Build Tools
│   ├── Makefile                    # Automated build system (nerdctl-based)
│   └── build/
│       └── Dockerfile              # Comprehensive toolbox image (50+ tools)
│
├── 📋 Templates
│   ├── templates/
│   │   ├── _helpers.tpl           # Helper functions
│   │   ├── serviceaccount.yaml    # ServiceAccount resource
│   │   └── deployment.yaml        # Deployment resource
│
├── 📜 Scripts
│   └── scripts/
│       └── deploy-offline.sh.template  # Offline deployment automation
│
├── 📂 Examples
│   └── examples/
│       ├── README.md              # Example configurations guide
│       ├── values-online.yaml     # Online deployment config
│       ├── values-offline.yaml    # Offline deployment config
│       ├── values-mongodb.yaml    # MongoDB namespace example
│       └── DEPLOYMENT-EXAMPLES.sh # 10 deployment scenarios
│
├── 🧪 Tests
│   └── tests/
│       ├── README.md              # Testing guide
│       ├── test-helm-chart.sh     # Automated test suite
│       └── TEST-RESULTS.md        # Validation results
│
└── 📦 Distribution (created by make)
    └── dist/
        ├── offline-bundle/        # Offline bundle contents
        │   ├── images/           # Container image tarballs
        │   ├── charts/           # Packaged Helm charts
        │   ├── scripts/          # Deployment scripts
        │   ├── docs/             # Documentation
        │   ├── README.txt        # Bundle quick start
        │   └── MANIFEST.txt      # Bundle manifest with checksums
        └── *.tar.gz              # Final bundle archive
```

---

**Status:** ✅ Production Ready  
**Runtime:** nerdctl + containerd  
**Tools:** 50+ pre-installed  
**Version:** 0.1.0  
**Tested:** ✅ Kubernetes 1.19+ with Helm 3.0+  
**Last Updated:** November 24, 2025

---

## 🚀 Quick Reference Card

```bash
# BUILD
make build-image      # Build toolbox with nerdctl
make test-image       # Verify all tools installed
make offline-bundle   # Create complete offline bundle

# DEPLOY
helm install my-toolbox . -n toolbox --create-namespace

# ACCESS
kubectl exec -n toolbox -it deploy/my-toolbox-ultimate-k8s-toolbox -- bash

# INSIDE POD
show-versions.sh                    # View all tools
mongosh --version                   # MongoDB shell
kubectl get pods -A                 # Kubernetes ops
dig service.svc.cluster.local       # DNS debug
openssl s_client -connect host:443  # TLS debug
```
**Version:** 0.1.0  
**Tested:** ✅ Kubernetes 1.x with Helm 3.19.2  
**Last Updated:** November 24, 2025
