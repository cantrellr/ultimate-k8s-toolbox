FROM ubuntu:24.04

# Metadata
LABEL maintainer="Platform Team"
LABEL description="Swiss-army knife for Kubernetes, MongoDB, Ops Manager, networking, storage, and cluster inspection - True Admin Workstation"
LABEL version="1.0.0"

# Prevent interactive prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive
ENV PATH="/usr/local/bin:${PATH}"

# ============================================================================
# SECTION 1: Base System + Core Utilities
# ============================================================================
RUN apt-get update && apt-get install -y \
    # Core utilities
    curl \
    wget \
    ca-certificates \
    gnupg \
    lsb-release \
    apt-transport-https \
    software-properties-common \
    # Text editors
    vim \
    nano \
    # Version control
    git \
    # System monitoring and debugging
    htop \
    less \
    psmisc \
    strace \
    procps \
    lsof \
    iotop \
    file \
    # Shell enhancements
    bash-completion \
    && rm -rf /var/lib/apt/lists/*

# ============================================================================
# SECTION 2: X.509 / TLS / Certificate Tools
# ============================================================================
RUN apt-get update && apt-get install -y \
    openssl \
    gnutls-bin \
    && rm -rf /var/lib/apt/lists/*

# ============================================================================
# SECTION 3: Networking Tools (Complete Debug Stack)
# ============================================================================
RUN apt-get update && apt-get install -y \
    dnsutils \
    iputils-ping \
    iproute2 \
    netcat-openbsd \
    tcpdump \
    traceroute \
    net-tools \
    telnet \
    nmap \
    iperf3 \
    && rm -rf /var/lib/apt/lists/*

# ============================================================================
# SECTION 4: Storage + File Transfer Tools
# ============================================================================
RUN apt-get update && apt-get install -y \
    nfs-common \
    zip \
    unzip \
    tar \
    gzip \
    rsync \
    openssh-client \
    && rm -rf /var/lib/apt/lists/*

# ============================================================================
# SECTION 5: MongoDB Client Stack (Complete Suite)
# ============================================================================

# Install MongoDB Shell (mongosh)
RUN MONGOSH_VERSION="2.3.7" && \
    ARCH=$(dpkg --print-architecture) && \
    wget -q https://downloads.mongodb.com/compass/mongodb-mongosh_${MONGOSH_VERSION}_${ARCH}.deb && \
    dpkg -i mongodb-mongosh_${MONGOSH_VERSION}_${ARCH}.deb || apt-get install -f -y && \
    rm mongodb-mongosh_${MONGOSH_VERSION}_${ARCH}.deb

# Install MongoDB Database Tools (mongodump, mongorestore, bsondump, mongostat, etc.)
RUN MONGO_TOOLS_VERSION="100.10.0" && \
    ARCH=$(dpkg --print-architecture) && \
    wget -q https://fastdl.mongodb.org/tools/db/mongodb-database-tools-ubuntu2404-x86_64-${MONGO_TOOLS_VERSION}.deb && \
    dpkg -i mongodb-database-tools-ubuntu2404-x86_64-${MONGO_TOOLS_VERSION}.deb || apt-get install -f -y && \
    rm mongodb-database-tools-ubuntu2404-x86_64-${MONGO_TOOLS_VERSION}.deb

# Verify MongoDB tools installation
RUN mongosh --version && \
    mongodump --version && \
    mongorestore --version && \
    echo "MongoDB client stack installed successfully"

# ============================================================================
# SECTION 6: Kubernetes Admin Tooling
# ============================================================================

# Install kubectl
RUN KUBECTL_VERSION="v1.31.4" && \
    ARCH=$(dpkg --print-architecture) && \
    curl -LO "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/${ARCH}/kubectl" && \
    chmod +x kubectl && \
    mv kubectl /usr/local/bin/

# Install Helm
RUN curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Install jq (JSON processor) - already in base but ensuring it's here
RUN apt-get update && apt-get install -y jq && rm -rf /var/lib/apt/lists/*

# Install yq (YAML processor)
RUN YQ_VERSION="v4.45.1" && \
    ARCH=$(dpkg --print-architecture) && \
    wget -qO /usr/local/bin/yq https://github.com/mikefarah/yq/releases/download/${YQ_VERSION}/yq_linux_${ARCH} && \
    chmod +x /usr/local/bin/yq

# Install envsubst (part of gettext-base)
RUN apt-get update && apt-get install -y gettext-base && rm -rf /var/lib/apt/lists/*

# ============================================================================
# SECTION 7: Storage Tools (Trident + NFS)
# ============================================================================

# Install tridentctl (NetApp Trident CLI)
RUN TRIDENT_VERSION="24.10.0" && \
    wget https://github.com/NetApp/trident/releases/download/v${TRIDENT_VERSION}/trident-installer-${TRIDENT_VERSION}.tar.gz && \
    tar -xzf trident-installer-${TRIDENT_VERSION}.tar.gz && \
    cp trident-installer/tridentctl /usr/local/bin/ && \
    chmod +x /usr/local/bin/tridentctl && \
    rm -rf trident-installer-${TRIDENT_VERSION}.tar.gz trident-installer

# ============================================================================
# SECTION 8: Python Environment (Scripting + Automation)
# ============================================================================

# Install Python
RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip \
    python3-venv \
    && rm -rf /var/lib/apt/lists/*

# Install essential Python packages
RUN pip3 install --no-cache-dir --break-system-packages \
    pymongo \
    kubernetes \
    pyyaml \
    requests \
    jinja2 \
    click

# Verify Python installation
RUN python3 --version && pip3 --version

# ============================================================================
# SECTION 9: CA Trust Integration (Optional)
# ============================================================================

# Create directory for custom CA certificates
RUN mkdir -p /usr/local/share/ca-certificates/custom

# Add CA trust update script for runtime mounting
RUN echo '#!/bin/bash' > /usr/local/bin/update-ca-trust.sh && \
    echo '# Update CA trust when custom certificates are mounted' >> /usr/local/bin/update-ca-trust.sh && \
    echo 'if [ -f /tls/ca.crt ]; then' >> /usr/local/bin/update-ca-trust.sh && \
    echo '    echo "Installing custom CA certificate..."' >> /usr/local/bin/update-ca-trust.sh && \
    echo '    cp /tls/ca.crt /usr/local/share/ca-certificates/custom/cluster-ca.crt' >> /usr/local/bin/update-ca-trust.sh && \
    echo '    update-ca-certificates' >> /usr/local/bin/update-ca-trust.sh && \
    echo '    echo "CA trust updated successfully"' >> /usr/local/bin/update-ca-trust.sh && \
    echo 'fi' >> /usr/local/bin/update-ca-trust.sh && \
    chmod +x /usr/local/bin/update-ca-trust.sh

# ============================================================================
# SECTION 10: User Setup + Workspace Configuration
# ============================================================================

# Create a non-root user (use uid 10000 to avoid conflicts)
RUN useradd -m -s /bin/bash -u 10000 toolbox && \
    mkdir -p /home/toolbox/.kube /home/toolbox/scripts && \
    chown -R toolbox:toolbox /home/toolbox

# Set up workspace
WORKDIR /workspace
RUN chown toolbox:toolbox /workspace

# Create comprehensive tool verification script
RUN echo '#!/bin/bash' > /usr/local/bin/show-versions.sh && \
    echo 'echo "=============================================="' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "  🚀 Ultimate K8s Admin Workstation 🚀"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "  Swiss-army knife for K8s & MongoDB Ops"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "=============================================="' >> /usr/local/bin/show-versions.sh && \
    echo 'echo ""' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "📦 MONGODB CLIENT STACK:"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "  • mongosh: $(mongosh --version | head -1 | awk \"{print \\$2}\")"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "  • mongodump: $(mongodump --version | grep version | awk \"{print \\$3}\")"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "  • mongorestore: $(mongorestore --version | grep version | awk \"{print \\$3}\")"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo ""' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "☸️  KUBERNETES ADMIN TOOLS:"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "  • kubectl: $(kubectl version --client --short 2>/dev/null | grep Client | awk \"{print \\$3}\" || echo \"ready\")"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "  • helm: $(helm version --short | cut -d+ -f1)"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "  • jq: $(jq --version)"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "  • yq: $(yq --version | awk \"{print \\$3}\")"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo ""' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "🔒 TLS / X.509 TOOLS:"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "  • openssl: $(openssl version | awk \"{print \\$2}\")"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "  • gnutls: $(certtool --version | head -1 | awk \"{print \\$2}\")"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo ""' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "🌐 NETWORKING TOOLS:"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "  • dig, nslookup, ping, traceroute"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "  • tcpdump, netcat, nmap, curl, wget"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo ""' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "💾 STORAGE TOOLS:"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "  • tridentctl: $(tridentctl version --client 2>/dev/null | grep Version | awk \"{print \\$2}\" || echo \"ready\")"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "  • nfs-common, rsync, git"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo ""' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "🐍 PYTHON ENVIRONMENT:"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "  • Python: $(python3 --version | awk \"{print \\$2}\")"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "  • Packages: pymongo, kubernetes, pyyaml, requests"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo ""' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "🛠️  SYSTEM TOOLS:"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "  • vim, nano, htop, less, strace, lsof"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "  • bash-completion enabled"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo ""' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "=============================================="' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "  Ready for cluster ops! 🎯"' >> /usr/local/bin/show-versions.sh && \
    echo 'echo "=============================================="' >> /usr/local/bin/show-versions.sh && \
    echo 'echo ""' >> /usr/local/bin/show-versions.sh && \
    chmod +x /usr/local/bin/show-versions.sh

# Switch to non-root user
USER toolbox

# Set default command
CMD ["/bin/bash", "-c", "show-versions.sh && tail -f /dev/null"]
