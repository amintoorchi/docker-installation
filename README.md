#!/bin/bash

# colors for pretty output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

echo -e "${GREEN}🚀 Starting Docker installation on Ubuntu...${NC}"

# Update system
echo -e "${YELLOW}📦 Updating package list...${NC}"
sudo apt update -y

# Install prerequisites
echo -e "${YELLOW}📦 Installing prerequisites...${NC}"
sudo apt install -y ca-certificates curl

# Create keyrings directory
echo -e "${YELLOW}🔑 Setting up Docker GPG key...${NC}"
sudo install -m 0755 -d /etc/apt/keyrings

# Download Docker GPG key
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Detect Ubuntu codename
UBUNTU_CODENAME=$(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
ARCHITECTURE=$(dpkg --print-architecture)

echo -e "${GREEN}✅ Detected: Ubuntu $UBUNTU_CODENAME, Architecture: $ARCHITECTURE${NC}"

# Add Docker repository
echo -e "${YELLOW}📝 Adding Docker repository...${NC}"
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: ${UBUNTU_CODENAME}
Components: stable
Architectures: ${ARCHITECTURE}
Signed-By: /etc/apt/keyrings/docker.asc
EOF

# Update package list with Docker repo
echo -e "${YELLOW}📦 Updating package list with Docker repo...${NC}"
sudo apt update -y

# Install Docker and plugins
echo -e "${YELLOW}🐳 Installing Docker...${NC}"
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Check if Docker installed successfully
if command -v docker &> /dev/null; then
    echo -e "${GREEN}✅ Docker installed successfully!${NC}"
    docker --version
    docker compose version
else
    echo -e "${RED}❌ Docker installation failed!${NC}"
    exit 1
fi

# Add current user to docker group (optional)
echo -e "${YELLOW}👤 Adding user to docker group...${NC}"
sudo usermod -aG docker $USER
echo -e "${GREEN}✅ User added to docker group. You may need to logout/login to use docker without sudo.${NC}"

# Enable and start Docker service
echo -e "${YELLOW}⚙️ Enabling Docker service...${NC}"
sudo systemctl enable docker
sudo systemctl start docker

echo -e "${GREEN}🎉 Docker installation complete!${NC}"
echo -e "${YELLOW}ℹ️  Tip: Run 'newgrp docker' or logout/login to use docker without sudo.${NC}"
