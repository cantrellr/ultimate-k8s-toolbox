# Ultimate K8s Tool

```
   ╔═══════════════════════════════════════════════════════════════════════╗
   ║                                                                       ║
   ║   █░█ █░░ ▀█▀ █ █▀▄▀█ ▄▀█ ▀█▀ █▀▀   █▄▀ ▄▀█ █▀   ▀█▀ █▀█ █▀█ █░░      ║
   ║   █▄█ █▄▄ ░█░ █ █░▀░█ █▀█ ░█░ ██▄   █░█ ▀▀█ ▄█   ░█░ █▄█ █▄█ █▄▄      ║
   ║                                                                       ║
   ║              ✈️  "First Flight" Release - v1.0.0  ✈️                 ║
   ╚═══════════════════════════════════════════════════════════════════════╝
```

# Ultimate Kubernetes Toolbox

**The comprehensive Kubernetes administration workstation**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Helm](https://img.shields.io/badge/Helm-3.x-blue.svg)](https://helm.sh)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.19+-326CE5.svg?logo=kubernetes&logoColor=white)](https://kubernetes.io)
[![GitHub release](https://img.shields.io/github/v/release/cantrellr/ultimate-k8s-toolbox)](https://github.com/cantrellr/ultimate-k8s-toolbox/releases)

*50+ pre-installed tools • Air-gapped ready • Multi-architecture (amd64/arm64)*

[Quick Start](#-quick-start) •
[Tools](#-included-tools) •
[Documentation](#-documentation) •
[Contributing](CONTRIBUTING.md)

</div>

---

## 🎯 Overview

**Ultimate K8s Toolbox** is a Helm chart that deploys a fully-equipped Kubernetes administration workstation directly into your cluster. Think of it as a Swiss Army knife pod — pre-loaded with everything you need for debugging, troubleshooting, and managing Kubernetes environments.

### Why Use This?

| Scenario | Solution |
|----------|----------|
| 🔍 Debug pod networking issues | `tcpdump`, `netcat`, `nmap`, `dig` all pre-installed |
| 🔐 Troubleshoot TLS/certificate problems | `openssl`, certificate verification tools, CA trust |
| 📊 Inspect MongoDB clusters | `mongosh`, `mongodump`, `mongostat` ready to go |
| ☸️ Manage Kubernetes resources | `kubectl`, `helm`, `k9s`, `stern` at your fingertips |
| 🚫 Work in air-gapped environments | Full offline deployment support with internal registries |
| 📋 Meet compliance requirements | SBOM generation, security scanning with `trivy` |

---

## 🏗️ Architecture

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                        KUBERNETES CLUSTER                                    │
│  ┌────────────────────────────────────────────────────────────────────────┐  │
│  │                         TOOLBOX NAMESPACE                              │  │
│  │                                                                        │  │
│  │   ┌─────────────────────────────────────────────────────────────────┐  │  │
│  │   │                    TOOLBOX POD                                  │  │  │
│  │   │  ┌───────────────────────┐  ┌────────────────────────────────┐  │  │  │
│  │   │  │    INIT CONTAINER     │  │       MAIN CONTAINER           │  │  │  │
│  │   │  │   update-ca-trust     │  │         toolbox                │  │  │  │
│  │   │  │  ─────────────────    │  │  ────────────────────────────  │  │  │  │
│  │   │  │  • Runs as root       │  │  • Runs as non-root (UID 1000) │  │  │  │
│  │   │  │  • Updates CA trust   │  │  • 50+ pre-installed tools     │  │  │  │
│  │   │  │  • Copies to volume   │  │  • kubectl, helm, k9s          │  │  │  │
│  │   │  │                       │  │  • mongosh, database clients   │  │  │  │
│  │   │  └───────────┬───────────┘  │  • Network debugging tools     │  │  │  │
│  │   │              │              │  • Python 3.12 + packages      │  │  │  │
│  │   │              ▼              │                                │  │  │  │
│  │   │  ┌───────────────────────┐  │                                │  │  │  │
│  │   │  │    SHARED VOLUME      │  │                                │  │  │  │
│  │   │  │   shared-ca-certs     │──┤  /etc/ssl/certs/               │  │  │  │
│  │   │  │   (emptyDir)          │  │                                │  │  │  │
│  │   │  └───────────────────────┘  └────────────────────────────────┘  │  │  │
│  │   │                                                                 │  │  │
│  │   │  ┌─────────────────────────────────────────────────────────┐    │  │  │
│  │   │  │                     VOLUMES                             │    │  │  │
│  │   │  │  • custom-ca-certs (Secret) - Your CA certificates      │    │  │  │
│  │   │  │  • workspace (emptyDir) - Working directory             │    │  │  │
│  │   │  │  • Custom volumes via values.yaml                       │    │  │  │
│  │   │  └─────────────────────────────────────────────────────────┘    │  │  │
│  │   └─────────────────────────────────────────────────────────────────┘  │  │
│  │                                                                        │  │
│  │   ┌────────────────────────────────────────────────────────────────┐   │  │
│  │   │                     SERVICE ACCOUNT                            │   │  │
│  │   │  • Optional ClusterRole for cluster-wide access                │   │  │
│  │   │  • Optional Role for namespace-scoped access                   │   │  │
│  │   │  • Configurable RBAC permissions                               │   │  │
│  │   └────────────────────────────────────────────────────────────────┘   │  │
│  └────────────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## ⚡ Quick Start

### Prerequisites

- Kubernetes 1.19+
- Helm 3.x
- kubectl configured for your cluster

### Online Deployment (5 minutes)

```bash
# Clone the repository
git clone https://github.com/cantrellr/ultimate-k8s-toolbox.git
cd ultimate-k8s-toolbox

# Deploy to your cluster
helm install toolbox ./chart -n toolbox --create-namespace

# Access the toolbox
kubectl exec -it -n toolbox deploy/toolbox-ultimate-k8s-toolbox -- bash
```

### Using the Quick Access Script

```bash
# Install the helper script
./scripts/install-toolbox.sh

# Now just run:
toolbox
```

---

## 📦 Included Tools

<details>
<summary><b>☸️ Kubernetes & Container Tools (15)</b></summary>

| Tool | Version | Description |
|------|---------|-------------|
| `kubectl` | 1.31.x | Kubernetes CLI |
| `helm` | 3.x | Kubernetes package manager |
| `k9s` | Latest | Terminal UI for Kubernetes |
| `kubectx/kubens` | Latest | Context and namespace switcher |
| `stern` | Latest | Multi-pod log tailing |
| `kustomize` | Latest | Kubernetes configuration customization |
| `k3d` | Latest | k3s in Docker |
| `kind` | Latest | Kubernetes in Docker |
| `istioctl` | Latest | Istio service mesh CLI |
| `linkerd` | Latest | Linkerd service mesh CLI |
| `argocd` | Latest | ArgoCD CLI |
| `flux` | Latest | Flux CD CLI |
| `velero` | Latest | Backup and restore CLI |
| `kubeseal` | Latest | Sealed Secrets CLI |
| `krew` | Latest | kubectl plugin manager |

</details>

<details>
<summary><b>☁️ Cloud Provider CLIs (3)</b></summary>

| Tool | Version | Description |
|------|---------|-------------|
| `aws` | 2.x | AWS CLI |
| `az` | Latest | Azure CLI |
| `gcloud` | Latest | Google Cloud SDK |

</details>

<details>
<summary><b>🗄️ Database Clients (5)</b></summary>

| Tool | Version | Description |
|------|---------|-------------|
| `mongosh` | Latest | MongoDB Shell |
| `mongodump/restore` | Latest | MongoDB backup tools |
| `psql` | Latest | PostgreSQL client |
| `mysql` | Latest | MySQL client |
| `redis-cli` | Latest | Redis client |

</details>

<details>
<summary><b>🌐 Network Tools (15)</b></summary>

| Tool | Description |
|------|-------------|
| `curl` / `wget` | HTTP clients |
| `dig` / `nslookup` / `host` | DNS tools |
| `netcat` (nc) | Network utility |
| `nmap` | Network scanner |
| `tcpdump` | Packet capture |
| `traceroute` / `mtr` | Route tracing |
| `ping` | ICMP testing |
| `telnet` | Telnet client |
| `iperf3` | Bandwidth testing |
| `ss` / `netstat` | Socket statistics |
| `ip` / `ifconfig` | Network configuration |
| `whois` | Domain lookup |
| `openssl` | TLS/SSL toolkit |

</details>

<details>
<summary><b>🔐 Security Tools (4)</b></summary>

| Tool | Description |
|------|-------------|
| `trivy` | Vulnerability scanner |
| `grype` | Vulnerability scanner |
| `syft` | SBOM generator |
| `openssl` | Certificate operations |

</details>

<details>
<summary><b>🛠️ Development Tools (15+)</b></summary>

| Tool | Description |
|------|-------------|
| `git` | Version control |
| `vim` / `nano` | Text editors |
| `jq` / `yq` | JSON/YAML processors |
| `fzf` | Fuzzy finder |
| `bat` | Better cat |
| `ripgrep` (rg) | Fast grep |
| `fd` | Fast find |
| `htop` | Process viewer |
| `tree` | Directory listing |
| `tmux` | Terminal multiplexer |
| `Python 3.12` | With pip and common packages |

</details>

<details>
<summary><b>💾 Storage & Backup Tools (5)</b></summary>

| Tool | Description |
|------|-------------|
| `rclone` | Cloud storage sync |
| `mc` | MinIO client |
| `restic` | Backup tool |
| `rsync` | File synchronization |
| `tridentctl` | NetApp Trident CLI |

</details>

---

## 📚 Documentation

| Document | Description |
|----------|-------------|
| [📖 QUICKSTART.md](QUICKSTART.md) | Get started in 5 minutes |
| [🔧 TOOLS-REFERENCE.md](TOOLS-REFERENCE.md) | Complete tool documentation with examples |
| [✈️ OFFLINE-DEPLOYMENT.md](OFFLINE-DEPLOYMENT.md) | Air-gapped deployment guide |
| [🏗️ MAKEFILE.md](MAKEFILE.md) | Build system documentation |
| [📋 SBOM.md](SBOM.md) | Software Bill of Materials info |
| [🐳 NERDCTL-GUIDE.md](NERDCTL-GUIDE.md) | Container runtime guide |
| [📝 CHANGELOG.md](CHANGELOG.md) | Version history |

---

## 🚀 Deployment Options

### Option 1: Online Deployment

For clusters with internet access:

```bash
helm install my-toolbox ./chart \
  -n toolbox --create-namespace \
  -f examples/values-online.yaml
```

### Option 2: Air-Gapped/Offline Deployment

For restricted environments without internet:

```bash
# 1. Build offline bundle (on machine with internet)
make offline-bundle

# 2. Transfer dist/offline-bundle/ to air-gapped environment

# 3. Run the deployment script
cd dist/offline-bundle
./scripts/deploy-offline.sh \
  --registry registry.internal:5000 \
  --namespace toolbox
```

See [OFFLINE-DEPLOYMENT.md](OFFLINE-DEPLOYMENT.md) for detailed instructions.

### Option 3: With Custom CA Certificates

For environments with internal PKI:

```bash
# Create CA secret
kubectl create secret generic toolbox-ca-certs \
  --from-file=root-ca.crt=/path/to/ca.crt \
  -n toolbox

# Deploy with CA enabled
helm install my-toolbox ./chart \
  -n toolbox \
  --set customCA.enabled=true \
  --set customCA.secretName=toolbox-ca-certs
```

---

## ⚙️ Configuration

### Key Values

| Parameter | Description | Default |
|-----------|-------------|---------|
| `image.repository` | Image repository | `ultimate-k8s-toolbox` |
| `image.tag` | Image tag | `1.0.0` |
| `global.imageRegistry` | Registry for offline deployments | `""` |
| `replicaCount` | Number of replicas | `1` |
| `serviceAccount.create` | Create ServiceAccount | `true` |
| `rbac.create` | Create RBAC resources | `true` |
| `rbac.clusterRole` | Create ClusterRole (vs Role) | `true` |
| `customCA.enabled` | Enable custom CA trust | `false` |
| `customCA.secretName` | Secret containing CA certs | `toolbox-ca-certs` |
| `resources.requests.cpu` | CPU request | `100m` |
| `resources.requests.memory` | Memory request | `256Mi` |
| `resources.limits.cpu` | CPU limit | `2` |
| `resources.limits.memory` | Memory limit | `4Gi` |

### Example values.yaml

```yaml
# Production offline deployment
global:
  imageRegistry: "harbor.internal.company.com"

image:
  repository: "platform/ultimate-k8s-toolbox"
  tag: "1.0.0"

imagePullSecrets:
  - name: harbor-credentials

customCA:
  enabled: true
  secretName: "company-ca-certs"

resources:
  requests:
    cpu: "200m"
    memory: "512Mi"
  limits:
    cpu: "4"
    memory: "8Gi"

securityContext:
  runAsNonRoot: true
  runAsUser: 1000
```

---

## 🔒 Security

### Container Security

- **Non-root by default**: Runs as UID 1000
- **No privilege escalation**: Disabled by default
- **Read-only root filesystem**: Supported (some tools require writeable dirs)
- **RBAC**: Configurable cluster/namespace-scoped permissions

### Reporting Vulnerabilities

Please report security vulnerabilities via [GitHub Security Advisories](https://github.com/cantrellr/ultimate-k8s-toolbox/security/advisories/new). See [SECURITY.md](SECURITY.md) for details.

---

## 🛠️ Building

### Prerequisites

- Docker or nerdctl/containerd
- Make
- Helm 3.x

### Build Commands

```bash
# Build image
make build

# Build multi-arch (amd64, arm64)
make build-multi

# Run tests
make test

# Create offline bundle
make offline-bundle

# Generate SBOM
make sbom

# See all targets
make help
```

---

## 📊 Project Structure

```
ultimate-k8s-toolbox/
├── build/
│   └── Dockerfile          # Container image definition
├── chart/
│   ├── Chart.yaml          # Helm chart metadata
│   ├── values.yaml         # Default configuration
│   └── templates/          # Kubernetes manifests
├── configs/                # Example configurations
├── examples/               # Deployment examples
│   ├── values-online.yaml
│   ├── values-offline.yaml
│   └── ...
├── scripts/
│   ├── deploy-offline.sh.template
│   ├── install-toolbox.sh
│   └── toolbox             # Quick exec helper
├── tests/                  # Test scripts
├── CHANGELOG.md            # Release history
├── CONTRIBUTING.md         # Contribution guide
├── LICENSE                 # MIT License
└── README.md               # This file
```

---

## 🤝 Contributing

Contributions are welcome! Please read our [Contributing Guide](CONTRIBUTING.md) and [Code of Conduct](CODE_OF_CONDUCT.md) first.

### Ways to Contribute

- 🐛 Report bugs via [GitHub Issues](https://github.com/cantrellr/ultimate-k8s-toolbox/issues)
- 💡 Suggest features via [GitHub Discussions](https://github.com/cantrellr/ultimate-k8s-toolbox/discussions)
- 🔧 Submit pull requests
- 📝 Improve documentation
- 🛠️ Request new tools

---

## 📜 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

- The Kubernetes community
- All the amazing open-source tool maintainers
- Contributors and users of this project

---

<div align="center">

**✈️ "First Flight" Release v1.0.0**

*Per aspera ad astra* — Through hardships to the stars

---

Made with ❤️ for the Kubernetes community

</div>
]]>