# ─────────────────────────────────────────────────────────────────────────────
# Base: Ubuntu 22.04 (.NET 6 SDK, multi-arch)
# ─────────────────────────────────────────────────────────────────────────────
FROM mcr.microsoft.com/devcontainers/dotnet:6.0-jammy

ARG DEBIAN_FRONTEND=noninteractive
USER root

# ─────────────────────────────────────────────────────────────────────────────
# Node 16  +  Gulp CLI
# ─────────────────────────────────────────────────────────────────────────────
ARG NODE_VERSION=16
RUN bash -c 'source /usr/local/share/nvm/nvm.sh && nvm install '"$NODE_VERSION"
RUN npm install --global gulp-cli

# ─────────────────────────────────────────────────────────────────────────────
# Azure CLI
# ─────────────────────────────────────────────────────────────────────────────
RUN curl -sL https://aka.ms/InstallAzureCLIDeb | bash

# ─────────────────────────────────────────────────────────────────────────────
# Docker CLI + Compose-v2
# ─────────────────────────────────────────────────────────────────────────────
RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
        gpg --dearmor -o /usr/share/keyrings/docker.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker.gpg] \
        https://download.docker.com/linux/ubuntu jammy stable" \
        > /etc/apt/sources.list.d/docker.list && \
    apt-get update && \
    apt-get install -y docker-ce-cli docker-compose-plugin && \
    rm -rf /var/lib/apt/lists/*

# ─────────────────────────────────────────────────────────────────────────────
# Non-root vscode user (already exists in base image)
# ─────────────────────────────────────────────────────────────────────────────
USER vscode
WORKDIR /home/vscode

# ─────────────────────────────────────────────────────────────────────────────
# Azure Pipelines agent 3.229.1 – auto-detect architecture
# ─────────────────────────────────────────────────────────────────────────────
ARG AGENT_VERSION=3.229.1
RUN mkdir -p /home/vscode/azure-pipelines && \
    cd       /home/vscode/azure-pipelines && \
    dpkgArch="$(dpkg --print-architecture)" && \
    case "$dpkgArch" in \
        amd64) agentArch="x64" ;; \
        arm64) agentArch="arm64" ;; \
        armhf) agentArch="arm" ;; \
        *) echo "Unsupported arch $dpkgArch"; exit 1 ;; \
    esac && \
    curl -sSL -o vstsagent.tar.gz \
         https://vstsagentpackage.azureedge.net/agent/${AGENT_VERSION}/vsts-agent-linux-${agentArch}-${AGENT_VERSION}.tar.gz && \
    tar xzf vstsagent.tar.gz && \
    ./bin/installdependencies.sh

COPY library-scripts/start.sh /home/vscode/azure-pipelines/start.sh
RUN chmod +x /home/vscode/azure-pipelines/start.sh
