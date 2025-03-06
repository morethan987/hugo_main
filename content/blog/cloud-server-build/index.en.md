---
title: Cloud Service Deployment
weight: -85
draft: true
description: A detailed log of the deployment of cloud service from scratch
slug: cloud-server-build
tags:
  - Cloud-Service
series:
  - AI-Project
series_order: 2
date: 2025-03-06
lastmod: 2025-03-06
authors:
  - Morethan
---
{{< lead >}}
Detailed documentation of deploying an LLM app from scratch, with a primary focus on setting up a cloud server environment. This document will be updated continuously...
{{< /lead >}}

## Introduction

Before deploying your LLM application, ensure you have completed:

1. A frontend framework with core functionality
2. A relatively complete backend code repository

This guide primarily records the setup of a cloud server for testing and development. It does not yet cover production environments—refer to other resources if needed, as I only document what I've actually practiced 🫡.

## Exploring Deployment Options

Although the title mentions "Cloud Server Setup," I'd also like to mention some alternatives. Not every test environment needs an expensive cloud server. If you've already decided on a cloud server, jump directly to: [Cloud Server Setup Steps]({{< relref "#cloud-server-setup-steps" >}})

All content below assumes you're working primarily on Windows since I've never used Linux or MacOS 😢.

### Local Server

If you're a truly independent developer, requiring no collaboration, you don't necessarily need a cloud server—a Docker environment is sufficient.

After installing Docker Desktop, you'll need to perform **container orchestration** based on your existing backend code:

1. Set up API endpoints: Develop core business logic in your preferred programming language.
2. Prepare necessary environment files: Such as `.env`, `pyproject.toml`, etc.
3. Create a Dockerfile: Essentially a set of instructions to initialize your container.
4. Write a `docker-compose.yml` file: Define images, internal networks, ports, and mounted volumes required by your application.

Run a single command:

```bash
docker-compose up --build
```

Once Docker initializes, access your services via `localhost`, replicating the experience of a cloud server.

### Port Forwarding

If you've built a Docker service locally but need a shared testing environment for teammates, simply forward your local ports.

There are many tools for port forwarding. Personally, I recommend [Sakura Frp](https://www.natfrp.com/?page=panel&module=download).

Port forwarding is convenient but unsuitable if you need secure **frontend-backend communication** 😢.

Initially, my idea was running the frontend locally, backend inside Docker, and forwarding frontend/backend services individually. However, lacking SSL certificates caused browsers to block cross-domain HTTPS requests due to security concerns 😢. Only my own computer could run the service successfully; other devices failed—even self-signed certificates were difficult to configure due to browser security restrictions 😭.

After unsuccessful attempts, I ultimately chose cloud servers.

### Cloud Server

Cloud servers are simple to set up nowadays but have one disadvantage: cost 😢.

For development/testing environments, you don’t need high-end servers. Choose based on your budget and team size.

I chose Tencent Cloud’s lightweight server:  
2 CPU cores, 2GB RAM, 4Mbps bandwidth, 50GB system disk, 300GB monthly traffic  
Mainly due to cost-effectiveness—only ¥88 for the first year.

## Cloud Server Setup Steps

### Domain Registration

### DNS Configuration

### Purchasing a Cloud Server

### Server Configuration

The server OS used here is Ubuntu Server 24.04 LTS 64bit. All commands are based on this system.

#### File Transfer

For transferring files from local to server, I recommend using [FileZilla](https://filezilla-project.org/download.php): graphical interface, user-friendly, perfect if you dislike memorizing command lines like me 😝.

Of course, using terminal commands (`rsync` or `scp`) also works fine.

{{< alert icon="pencil" cardColor="#1E3A8A" textColor="#E0E7FF" >}}

{{< /alert >}}
#### Installing Docker

Execute each of the following commands, including detailed comments:

```bash
sudo apt-get update # Upgrade packages

sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release # Install dependencies for HTTPS and verification

# Install Docker’s official GPG key to ensure authenticity of Docker packages
sudo curl -fsSL https://mirrors.cloud.tencent.com/docker-ce/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

# Add Docker’s official repository to the APT source list
sudo install -m 0755 -d /etc/apt/keyrings

sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch={{< katex >}}\\((dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://mirrors.cloud.tencent.com/docker-ce/linux/ubuntu/ \\\\)(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update package index to include Docker repository
sudo apt-get update

# Install Docker Engine and related tools
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Verify Docker installation information
sudo docker info

# Start Docker service
sudo systemctl start docker

# Enable Docker to start at boot
sudo systemctl enable docker

# Verify Docker service status
sudo systemctl status docker
```

{{< alert icon="triangle-exclamation" cardColor="#ffcc00" textColor="#333333" iconColor="#8B6914" >}}

{{< /alert >}}
#### Using Docker

```bash
sudo systemctl start docker # Start Docker service

# Configure Docker mirror (Tencent Cloud source)
sudo vim /etc/docker/daemon.json # Create Docker config file

# Press 'I', enter insert mode, then input:
{
   "registry-mirrors": [
   "https://mirror.ccs.tencentyun.com"
  ]
}

# Press 'esc', then type ':wq' to save and exit Vim

sudo systemctl restart docker # Restart Docker service to apply config

# Verify Docker source settings (should display Registry Mirrors URL)
sudo docker info

# Pull a test image
sudo docker pull nginx
```

### Miscellaneous

## References

- [Logging into a Linux instance using a third-party SSH terminal - Tencent Cloud](https://cloud.tencent.com/document/product/1207/44578)
- [Logging into a Linux instance using local SSH terminal - Tencent Cloud](https://cloud.tencent.com/document/product/1207/44643)
- [Installing Docker on Linux](https://blog.csdn.net/weixin_48953586/article/details/145597723)
- [Ubuntu 22.04 Docker Installation](https://blog.csdn.net/weixin_44355653/article/details/140267707)
- [Purpose of Docker's official GPG Key](https://blog.csdn.net/qq_36777143/article/details/144872919)
- [Cloud Server Docker Setup - Tencent Cloud](https://cloud.tencent.com/document/product/213/46000#1H-kXbk9zoqvzYMVPVsBO)
- [Docker Installation and Mirror Acceleration - Tencent Cloud](https://cloud.tencent.com/document/product/1207/45596)
