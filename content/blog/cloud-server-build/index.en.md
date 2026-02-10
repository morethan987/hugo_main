---
title: Cloud Service Deployment
weight: -80
draft: false
description: Detailed documentation of the cloud service platform setup process from scratch
slug: cloud-server-build
language: en
tags:
  - Cloud-Service
series:
  - Technical Miscellany
series_order: 7
date: 2025-03-06
lastmod: 2025-03-16
authors:
  - Morethan
---

{{< lead >}} Detailed documentation of the process of deploying an LLM application from scratch, focusing mainly on the initial setup of cloud servers, continuously updated... {{< /lead >}}

## Preface

Before deploying your LLM application, you should have already completed:

1. A web front-end framework with core functionalities
2. A relatively complete back-end code repository

This document mainly records the setup process of cloud services for testing and development environments. Cloud servers for production environments are not covered here, so please refer to other articles. No practice, no say🫡

## Solution Exploration

Although this article is about "Cloud Server Setup," I still want to document some "non-cloud service" solutions, as not all testing and development environments require an expensive cloud server. If you have already decided to use a cloud server, then skip to: [Cloud Service Operation Process]({{< relref "#cloud-service-operation-process" >}})

The following content assumes your main production environment is Windows, as I have not used Linux or MacOS😢

### Local Services

#### General Idea

If you are a truly "independent" developer with no need for collaborative development, then you don't need a cloud server at all. You just need Docker.

After installing Docker Desktop, you need to perform **container orchestration** based on your existing back-end service code.

This step mainly involves:

1. Establishing API interfaces: Writing the main operational logic, which can be done in any programming language you prefer
2. Creating necessary environment files: Such as `.env`, `pyproject.toml`, etc.
3. Creating a DockerFile: Essentially a set of command-line instructions to initialize your container
4. Creating a `docker-compose.yml` file: In this file, you need to orchestrate the images used by your app, specifying internal networks, ports, mounted volumes, and other necessary settings

Then, run the command: `docker-compose up --build`

After Docker initialization, you can directly access your service via the `localhost` domain, just like using a cloud server.

#### Docker Path Issues

When orchestrating Docker containers, there are several necessary **working path parameters** to understand, otherwise, you might encounter "file not found" errors😢

1. `build.context`: This parameter exists in `docker-compose.yml` and refers to the "build context," pointing to a real directory on your local machine
2. `WORKDIR`: This parameter exists in the `Dockerfile` and is used to specify the "working directory"; subsequent RUN, COPY, and other commands will execute in this directory if **relative paths** are used, but absolute path parameters are not affected

### Port Forwarding

If you have built a Docker service locally but your team members need a unified testing environment, you can simply perform port forwarding on your local service.

There are many applications that support port forwarding, and I won't list them all here. I personally use [Sakura Frp](https://www.natfrp.com/?page=panel&module=download)

Although port forwarding is very convenient, if you need **front-end and back-end interaction**, it's best not to use port forwarding😢

My initial idea was: run the front-end web page locally, and also run the back-end service in a local Docker container, then use port forwarding to forward the front-end web page to users and the back-end to the front-end.

It seemed fine, but since your network service is not SSL verified, it cannot send https requests, causing the browser to block cross-origin insecure resource requests from the front-end to the back-end😢 The result is that only your computer can run the complete service process, and other devices will fail; even if you configure a self-signed certificate, it's hard to pass the client browser's security mechanism😭

After unsuccessful attempts, I chose to use a cloud server.

### Cloud Services

Now, the operation process of cloud servers is very simple, but there is one drawback: expensive😢

Of course, if it's not for a production environment but a testing and development environment, you don't need to choose a top-tier server. Choose a suitable one based on your financial situation and team size.

I ultimately chose Tencent Cloud's lightweight server: 2 cores + 2GB RAM + 4M bandwidth + 50GB system disk + 300GB monthly traffic

No other reason, mainly because it's cheap, only ￥88 for the first year.

## Cloud Service Operation Process

### Register a Domain

There are plenty of tutorials online, so I won’t go into detail here. Just pick a domain registrar you trust, and you’re good to go.

Not sure what a domain name is? Here’s my simple way of putting it: Do you remember your apartment’s full official address? Probably not. But you definitely remember the name of your apartment complex, right? A domain name is kind of like that—easy to remember.

Of course, if you’re a big company, it’s also about looking impressive 😄

### Set Up DNS Resolution

DNS resolution is usually a service provided by your domain registrar. When you buy a domain, they’ll typically throw in a free DNS resolution service. It’s a basic version, sure—but hey, it works👌

So what is DNS resolution exactly? Think of a domain name like the name of your apartment complex. It makes perfect sense to you, but outsiders probably have no idea where that is. DNS acts like a translator—it turns your apartment complex's name into the full official address: Province XX, City XX, Street XX...

That way, even people from out of town can find your place and drop by without getting lost.

### Purchase a Cloud Server

Just buy it from a cloud server provider—this service system is already well-established. Nothing more to say. 😅  

### Configure the Server

The server operating system I use here is Ubuntu Server 24.04 LTS 64bit, and subsequent commands are based on this system.

#### File Transfer

To transfer files from the local machine to the server, I chose to use some more modern terminals, such as: [Tabby](https://tabby.sh/), [WindTerm](https://github.com/kingToolbox/WindTerm), and [Warp](https://www.warp.dev/), etc.

I randomly chose Tabby, which makes remote connections more convenient and has built-in SFTP for easy file transfer. If you have the energy to customize the terminal interface, Tabby can be very aesthetically pleasing😄

Of course, more traditional solutions include using [FileZilla](https://filezilla-project.org/download.php); and if you don't mind the hassle, you can directly use terminal commands: use `rsync` or `scp` commands.

> [!NOTE] Note
> If you are using a Tencent Cloud server and are curious about what the `lighthouse` folder is🤨: this folder is the account for one-click password-free login.

#### Install Docker

Execute the following commands one by one, each command has an explanation:

```bash
sudo apt-get update # Upgrade

sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release # Install dependency tools, mainly https transmission and verification-related toolkits

# Install Docker's GPG key to ensure the Docker image you download has not been tampered with
sudo curl -fsSL https://mirrors.cloud.tencent.com/docker-ce/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

# Add Docker's official software repository to the system's APT source list
# I don't think you want to know what this bunch means٩(•̤̀ᵕ•̤́๑)
sudo install -m 0755 -d /etc/apt/keyrings

sudo chmod a+r /etc/apt/keyrings/docker.asc

echo   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://mirrors.cloud.tencent.com/docker-ce/linux/ubuntu/ \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" |   sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update to let apt recognize the newly added Docker software source
sudo apt-get update

# Install Docker engine
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Start Docker
sudo docker info                   # Verify installation information
sudo systemctl start docker        # Start service
sudo systemctl enable docker       # Enable auto-start on boot
sudo systemctl status docker       # Verify service status

```

> [!warning] Warning
> After running the last command, you will enter a paged log view mode, similar to a Vim editor. Enter `:q` to return to the regular command line.

#### Use Docker

```bash
sudo systemctl start docker # Start service

# Configure Tencent Cloud's Docker source
sudo vim /etc/docker/daemon.json # Create configuration file

# Press the I key to switch to I mode and enter
{
   "registry-mirrors": [
   "https://mirror.ccs.tencentyun.com"
  ]
}

# Press esc, then enter ':wq', this is a Vim operation, meaning save and exit

sudo systemctl restart docker # Restart Docker service

sudo systemctl stop docker # Stop Docker service

# Verify source information, you should see the Registry Mirrors value is the URL you set
sudo docker info

# Pull image
sudo docker pull nginx

# If you want to use docker-compose
# On Ubuntu, use: docker compose
sudo docker compose up --build

# Start service without rebuilding
sudo docker compose up -d

# Stop all services
sudo docker compose down
```

> [!warning] Warning
> Before using the `docker compose` command, please create `docker-compose.yml` and `Dockerfile` according to the official tutorial.

#### Configure pip/poetry

Strangely, Tencent Cloud installs Python by default at the system level but does not install pip🤔 So you need to manually install it. Before installation, please confirm whether Python is installed.

> [!WARNING] Warning
> If you are not using pip at the system level but pip inside a Docker container, then changing the pip source at the system level will not have any effect.

Change pip source at the system level:

```bash
# Check Python version
python3 -V

# Check the path of python3
which python3

# Confirm Python is installed and pip is not installed
sudo apt install python3-pip

# If using Tencent's server, you can use the intranet domain for faster speed
pip config set global.index-url https://mirrors.tencentyun.com/pypi/simple

# If not, use the external domain
pip config set global.index-url https://mirrors.cloud.tencent.com/pypi/simple
```

Change pip source inside Docker container: Directly write the following command in your `Dockerfile`

```bash
# Configure pip source
# Use Tencent Cloud mirror source, note this is the intranet domain
RUN pip config set global.index-url https://mirrors.tencentyun.com/pypi/simple \
    && pip config set global.trusted-host mirrors.tencentyun.com
```

If you use poetry at the system level:

```bash
# Install poetry using online script
curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python

# Install poetry using apt
sudo apt install python3-poetry

# Use Tencent Cloud mirror source, note this is the intranet domain
sudo poetry config repositories.tencentyun https://mirrors.tencentyun.com/pypi/simple
```

> [!NOTE] Note
> If your system does not have pip installed by default, it is not recommended to install pip first and then install poetry.

#### SSL/TLS

SSL/TLS is a set of encrypted transmission protocols, carefully designed by cryptography experts. The widely used **Hypertext Transfer Protocol Secure** (HTTPS) is obtained by encrypting the plaintext transmission protocol HTTP using SSL/TLS.

A core part of HTTPS is the handshake before data transmission, during which the password for data encryption is determined. During the handshake, the website sends an SSL certificate to the browser. The SSL certificate is similar to our daily ID card, serving as an identity proof for HTTPS websites. The SSL certificate contains the website's domain name, certificate validity period, certificate issuing authority, and the public key used for encrypting the transmission password. Before generating the password, the browser needs to verify whether the currently accessed domain name matches the domain name bound to the certificate, and also verify the **Certificate Authority** (CA). If the verification fails, the browser will give a certificate error prompt.

Generally, SSL certificates need to be purchased from CA institutions, but some CA institutions provide free SSL certificate services, such as: [Let’s Encrypt](https://letsencrypt.org/)

After long-term development, the certificates issued by Let’s Encrypt can now be automatically processed through Certbot, and the certificates can be automatically re-applied when they expire. Of course, such convenience comes at a cost😢: You need to configure certbot, which is quite troublesome.

Here, I chose to directly deploy a certbot inside a Docker container to apply for SSL certificates, meaning you can directly modify the content in `docker-compose.yml` to pull the official certbot image.

```bash
# First, stop all Docker services
sudo docker compose down

# Start all services
sudo docker compose up -d

# Start part of the services
sudo docker compose up nginx -d

# View logs of a specific image
sudo docker logs server-app-1

# View logs of a specific service
sudo docker compose logs certbot-init

# Use recursive deletion to clear cache
rm -rf ./certbot/conf/*

# Enter a container's shell
sudo docker exec -it server-app-1 /bin/sh

# Remove a Docker image forcefully
sudo docker rmi -f server-frontend

# Restart a specific service in Docker Compose
sudo docker compose restart app

# Clear npm cache forcefully
npm cache clean --force

# Activate a Python virtual environment
source your_venv_name/bin/activate
```

### Filing

Filing using a corporate entity is not difficult, so I won't elaborate too much here. This section mainly discusses **individual entity** filing. Main reference: [Tencent Cloud Server Filing Full Process 40 Days of Filing Blood and Tears - Blog Garden](https://www.cnblogs.com/yyzwz/p/13393223.html)

{{< mermaid >}}
graph LR;

p1(Server Platform Real-name Authentication)-->p2(Real-name Authentication Purchase Domain)-->p3(Apply for Filing)-->p4(Platform Review)-->p5(Regulatory Authority Review)-->p6(Public Security Filing)
{{< /mermaid >}}

If you are a Tencent Cloud user, you can use the **Tencent Cloud Website Filing Mini Program**; after submitting the application, the platform will first review it, usually taking about 8 hours; then the platform will submit it to the regulatory authority for review, usually taking about 7 working days; after the regulatory authority review is passed, you need to complete the public security filing within 30 working days.

### Miscellaneous

This section mainly documents issues that may arise during the entire operation process.

#### Certbot Access Denied?

First, of course, check the network reasons. Check whether the security group of the server has opened ports 443 and 80, and check whether the firewall of the server operating system has blocked port access.

```bash
# Ubuntu installs ufw firewall by default
# Check ufw status, inactive status means not started, which is the ideal state
sudo ufw status
```

If you're based in China, here's another issue: Certbot can't reach your server if you don't have ICP备案 (ICP license). 😢

## References

- [Using Third-party SSH Terminal to Log in to Linux Instance - Operation Guide - Tencent Cloud](https://cloud.tencent.com/document/product/1207/44578)
- [Using Local Built-in SSH Terminal to Log in to Linux Instance - Operation Guide - Tencent Cloud](https://cloud.tencent.com/document/product/1207/44643)
- [Installing Docker in Linux Environment](https://blog.csdn.net/weixin_48953586/article/details/145597723)
- [Ubuntu 22.04 Install Docker](https://blog.csdn.net/weixin_44355653/article/details/140267707)
- [What is Docker's Official GPG Key For?](https://blog.csdn.net/qq_36777143/article/details/144872919)
- [Cloud Server Setup Docker - Practice Tutorial - Tencent Cloud](https://cloud.tencent.com/document/product/213/46000#1H-kXbk9zoqvzYMVPVsBO)
- [Installing Docker and Configuring Mirror Acceleration Source - Practice Tutorial - Tencent Cloud](https://cloud.tencent.com/document/product/1207/45596)
- [Debian 12 / Ubuntu 24.04 Install Docker and Docker Compose Tutorial - Shaobing Blog](https://u.sb/debian-install-docker/)
- [Step-by-Step Guide to Python pip Source Change Operation in Linux System - Tencent Cloud](https://cloud.tencent.com/developer/article/1635489)
- [pip Source Configuration - Tencent Cloud Developer Community - Tencent Cloud](https://cloud.tencent.com/developer/article/1601851)
- [How to Change Domestic Source for Poetry - Data Science SourceResearch](https://www.resourch.com/archives/66.html)
- [Linux Stop Docker Container: Single, Multiple, or All](https://cn.linux-console.net/?p=20047)
- [Where to View Docker Logs? How to View Logs in Linux Server - CSDN Blog](https://blog.csdn.net/qq_35393472/article/details/136719510)
- [Troubleshooting and Solving Linux Firewall Closed but Still Unable to Access Web](https://cloud.baidu.com/article/2693474)
- [About Ubuntu Console Opened All Ports, but External Hosts Still Cannot Access Corresponding Port Services - CSDN Blog](https://blog.csdn.net/weixin_51125521/article/details/132273384?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-132273384-blog-106063657.235%5Ev43%5Econtrol&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-132273384-blog-106063657.235%5Ev43%5Econtrol&utm_relevant_index=2)
- [Three Minutes to Explain SSL Authentication and Encryption Technology - CSDN Blog](https://blog.csdn.net/qq_16481211/article/details/119860572)
- [One Article to Thoroughly Understand SSL/TLS Protocol - Zhihu](https://zhuanlan.zhihu.com/p/133375078)
- [HTTPS and SSL Certificate Overview | Rookie Tutorial](https://www.runoob.com/w3cnote/https-ssl-intro.html)
- [Four Common Free Certificate Application Methods - CSDN Blog](https://blog.csdn.net/weixin_45444133/article/details/120900424)
- [Using Docker to Deploy Nginx and Configure HTTPS - TandK - Blog Garden](https://www.cnblogs.com/tandk-blog/p/15449873.html)
- [Using Docker + Nginx + Certbot to Automate SSL Certificate Management - CSDN Blog](https://blog.csdn.net/hjj19930729/article/details/145384756)
- [The Certificate Authority Failed to Download the Temporary Challenge Files Created by Certbot -- Connection Refused - Help - Let's Encrypt Community Support](https://community.letsencrypt.org/t/the-certificate-authority-failed-to-download-the-temporary-challenge-files-created-by-certbot-connection-refused/159426/26)
- [ICP Filing First Filing - Tencent Cloud](https://cloud.tencent.com/document/product/243/97668)
- [ICP Filing Requirements by Province - Tencent Cloud](https://cloud.tencent.com/document/product/243/3474)
- [Complete Tencent Cloud Server Filing Process: 40 Days of Painful Experience - Zheng Weizhong - CNBlogs](https://www.cnblogs.com/yyzwz/p/13393223.html)
- [ICP Filing and Public Security Filing Process - Tencent Cloud](https://cloud.tencent.com/document/product/243/19142)
- [ICP Filing Video Verification - Tencent Cloud](https://cloud.tencent.com/document/product/243/34945)
- [In-Depth Analysis of npm Cache: Understanding, Usage, and Clearing Guide_npm Clear Cache - CSDN Blog](https://blog.csdn.net/2401_85762266/article/details/140127881)
- [Deploying a Web Application with Docker - Zhihu](https://zhuanlan.zhihu.com/p/26418829)
- [Why Do HTTPS Requests Get Downgraded to HTTP? - CSDN Blog](https://blog.csdn.net/weixin_43947690/article/details/88295278)
- [HTTPS Redirected to HTTP—Risk Control-CSDN Blog](https://blog.csdn.net/zhuye1992/article/details/80496151?spm=1001.2101.3001.6650.4&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-4-80496151-blog-88295278.235%5Ev43%5Econtrol&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-4-80496151-blog-88295278.235%5Ev43%5Econtrol&utm_relevant_index=9)
- [How to Fill Out the Public Security Network Filing — Guide to Public Security Network Filing Information_Filing Service - Alibaba Cloud Help Center](https://help.aliyun.com/zh/icp-filing/basic-icp-service/the-public-security-network-for-the-record-information-fill-in-the-guide#:~:text=%E8%8B%A5%E6%82%A8%E5%8A%9E%E7%90%86%E5%85%AC%E5%AE%89%E8%81%94%E7%BD%91%E5%A4%87%E6%A1%88%E7%9A%84%E5%9F%9F%E5%90%8D%E6%98%AF%E9%80%9A%E8%BF%87%E9%98%BF%E9%87%8C%E4%BA%91%E6%B3%A8%E5%86%8C%EF%BC%8C%E5%8D%95%E5%87%BB%20%E6%9F%A5%E8%AF%A2%E7%BD%91%E7%BB%9C%E6%B3%A8%E5%86%8C%E6%9C%8D%E5%8A%A1%E5%95%86%E3%80%82%20%E6%A0%B9%E6%8D%AE%E5%A6%82%E4%B8%8B%E9%98%BF%E9%87%8C%E4%BA%91%E4%BF%A1%E6%81%AF%E5%A1%AB%E5%86%99%E3%80%82%20%E8%AF%B7%E6%A0%B9%E6%8D%AE%E7%BD%91%E7%AB%99%E5%AE%9E%E9%99%85%E5%BC%80%E5%B1%95%E7%9A%84%E4%B8%9A%E5%8A%A1%E9%A1%B9%E7%9C%9F%E5%AE%9E%E9%80%89%E6%8B%A9%EF%BC%8C%E4%BE%8B%E5%A6%82%EF%BC%9A%E7%BD%91%E7%BB%9C%E5%9F%BA%E7%A1%80%E7%B1%BB%20A%E3%80%81%E7%BD%91%E7%BB%9C%E9%94%80%E5%94%AE%E7%B1%BB%20B%E3%80%81%E7%94%9F%E6%B4%BB%E6%9C%8D%E5%8A%A1%E7%B1%BB%20C%20%E5%92%8C%E7%A4%BE%E4%BA%A4%E6%96%87%E5%A8%B1%E7%B1%BB,D%20%E7%AD%89%E3%80%82%20%E5%A6%82%E6%9E%9C%E4%B8%8D%E6%B6%89%E5%8F%8A%EF%BC%8C%E5%B0%86%20%E6%98%AF%E5%90%A6%E6%8F%90%E4%BE%9B%E4%BA%92%E8%81%94%E7%BD%91%E4%BA%A4%E4%BA%92%E6%9C%8D%E5%8A%A1%20%E9%80%89%E6%8B%A9%E4%B8%BA%20%E5%90%A6%E3%80%82%20%E5%A6%82%E6%9E%9C%E6%B6%89%E5%8F%8A%EF%BC%8C%E6%A0%B9%E6%8D%AE%E7%BD%91%E7%AB%99%E5%AE%9E%E9%99%85%E5%BC%80%E5%B1%95%E7%9A%84%E4%B8%9A%E5%8A%A1%E9%A1%B9%E7%9C%9F%E5%AE%9E%E9%80%89%E6%8B%A9%E3%80%82%20%E6%98%AF%E5%90%A6%E6%8F%90%E4%BE%9B%E6%B6%89%E5%8F%8A%E7%AE%A1%E5%88%B6%E7%89%A9%E5%93%81%E4%BF%A1%E6%81%AF%E5%8F%91%E5%B8%83%E6%9C%8D%E5%8A%A1%EF%BC%9A%E8%AF%B7%E6%8C%89%E7%85%A7%E7%BD%91%E7%AB%99%E6%B6%89%E5%8F%8A%E8%8C%83%E5%9B%B4%E5%A6%82%E5%AE%9E%E5%8B%BE%E9%80%89%E3%80%82)
