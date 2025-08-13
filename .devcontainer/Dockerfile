# Space-optimized Python devcontainer for Agentic IDE Template
FROM mcr.microsoft.com/devcontainers/python:3.12

# Install essential system dependencies with minimal footprint
RUN apt-get update && apt-get install -y --no-install-recommends \
    git \
    curl \
    wget \
    ca-certificates \
    && curl -fsSL https://deb.nodesource.com/setup_lts.x | bash - \
    && apt-get install -y nodejs \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean

# Configure vscode user permissions (user already exists in base image)
RUN usermod -aG sudo vscode || true && \
    echo "vscode ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Set up Python environment optimizations
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

# Switch to vscode user
USER vscode
WORKDIR /workspaces

# Ensure proper permissions for workspace
RUN sudo chown -R vscode:vscode /workspaces

# Pre-install common development tools to avoid repeated installations
RUN pip install --no-cache-dir --user \
    pip-tools \
    wheel \
    setuptools

# Set default shell
SHELL ["/bin/bash", "-c"]