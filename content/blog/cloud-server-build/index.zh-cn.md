---
title: 云服务搭建
weight: -85
draft: true
description: 详细记录了从零开始的云服务平台搭建过程
slug: cloud-server-build
tags:
  - 云服务
series:
  - AI工程
series_order: 2
date: 2025-03-06
lastmod: 2025-03-06
authors:
  - Morethan
---
{{< lead >}}
详细记录了从零开始发布 LLM 应用的过程，主要内容关注云服务器的初始化搭建过程，持续更新中...
{{< /lead >}}

## 前言
在发布你的 LLM 应用之前，你应该已经基本完成了：
1. 带有核心功能的网页前端框架
2. 功能相对完整的后端代码仓库

这里主要记录了测试开发环境的云服务搭建过程，用于生产环境的云服务器目前还没有涉及，因此请参考别的文章，没有实践就没有发言权🫡

## 方案探索
虽然这篇文章是"云服务器搭建"，但是我还是想记录一些"非云服务"的方案，毕竟不是所有的测试开发环境都需要一个昂贵的云服务器。如果你已经确定了要使用云服务器，那么跳转到：[云服务操作流程]({{< relref "#云服务操作流程" >}})

下面的内容均假定你的主力生产环境是 Windows，因为我 Linux 和 MacOS 都没用过😢

### 本地服务
如果你是一个真正意义上的"独立"开发者，没有任何人需要与你进行协同开发，那么你完全不需要一个云服务器，你只需要一个 Docker 就行。

安装 DockerDesktop 之后，你需要根据你已有的后端服务代码进行**容器编排**

这一步主要是：

1. 建立 API 接口：编写主操作逻辑，这里的主操作逻辑可以使用任何你喜欢的编程语言来编写
2. 创建必要的环境文件：例如 `.env` `pyproject.toml` 等环境文件
3. 创建 DockerFile：其实就是一个命令行指令集，用来初始化你的容器
4. 创建 `docker-compose.yml` 文件：在这个文件中你需要去编排你的 app 用到的镜像，指定内部网络、端口、挂载卷等等必要设置

然后一个命令：`docker-compose up --build`

Docker 初始化完成之后，你就可以直接通过 `localhost` 这个域名来访问你的服务了，就和使用云服务器的效果一样。

### 端口转发
如果你在自己的本地构建了一个 Docker 服务，但是你的团队成员又需要有一个统一的测试环境，那么直接在你的本地服务的基础上进行端口转发就行了。

能够支持端口转发的应用很多，这里不一一列举了，本人使用的是[Sakura Frp](https://www.natfrp.com/?page=panel&module=download)

虽然直接进行端口转发非常方便，但是如果你需要进行**前后端的交互**，那么最好不要使用端口转发😢

我原本的思路是：前端网页跑在本地，然后后端服务也跑在本地的 Docker 里面，然后使用端口转发，把前端网页转发给用户，把后端转发给前端。

看上去没什么问题，但是由于你的网络服务没有经过 SSL 验证，无法发送 https 请求，就导致浏览器阻止了前端向后端进行跨域的不安全资源请求😢最后的结果就是，只有你的电脑能够跑通完整的服务流程，别设备都会失败；即便配置了自签名证书，也很难通过客户端的浏览器的安全机制😭

最后尝试未果，选择使用云服务器。

### 云服务
现在云服务器的操作流程已经非常简单了，但是有一个缺点：贵😢

当然，如果不是生成环境而是测试开发环境的话，也不用选择特别顶尖的服务器，根据自身财力和团队成员数量，选择一个合适的就行

我最终选择的是腾讯云的轻型服务器： 2核+2G运存+4M带宽+50G系统盘+300G月流量

没有别的理由，主要就是便宜，首年才 ￥88

## 云服务操作流程
### 注册域名

### 设置 DNS 解析

### 购买云服务器

### 配置服务器
我这里使用的服务器操作系统是 Ubuntu Server 24.04 LTS 64bit，后续操作命令也是基于这个系统

#### 文件传输
为了从本地传输文件到服务区上，我选择使用 [FileZilla](https://filezilla-project.org/download.php)：图形化界面，傻瓜式操作，适合不想动脑子背命令的人，比如我😝

当然，直接用终端命令也是可以的：使用 `rsync` 或者 `scp` 命令就行

{{< alert icon="pencil" cardColor="#1E3A8A" textColor="#E0E7FF" >}}
如果你用的是腾讯云的服务器，同时你也好奇 `lighthouse` 文件夹是什么🤨：这个文件夹就是一键免密登录的账户
{{< /alert >}}
#### 安装Docker
逐条执行下面的命令即可，每条命令都有说明：

```bash
sudo apt-get update # 升级

sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release # 安装依赖工具，主要是https传输和验证相关的工具包

# 安装Docker的GPG密钥，为了确保你下载的Docker镜像没有被篡改
sudo curl -fsSL https://mirrors.cloud.tencent.com/docker-ce/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

# 将Docker的官方软件仓库添加到系统的APT源列表中
# 我觉得你不会想知道这一堆到底是什么意思٩(•̤̀ᵕ•̤́๑)
sudo install -m 0755 -d /etc/apt/keyrings

sudo chmod a+r /etc/apt/keyrings/docker.asc

echo   "deb [arch={{< katex >}}\\((dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://mirrors.cloud.tencent.com/docker-ce/linux/ubuntu/ \\\\)(. /etc/os-release && echo "$VERSION_CODENAME") stable" |   sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 更新一下，让apt能够识别新添加的Docker软件源
sudo apt-get update

# 安装Docker引擎
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 启动Docker
sudo docker info                   # 验证安装信息
sudo systemctl start docker        # 启动服务
sudo systemctl enable docker       # 开机自启
sudo systemctl status docker       # 验证服务状态

```

{{< alert icon="triangle-exclamation" cardColor="#ffcc00" textColor="#333333" iconColor="#8B6914" >}}
运行完最后一条指令，会进入分页查看日志模式，类似于一个 Vim 编辑器，输入 `:q` 回到常规命令行
{{< /alert >}}
#### 使用Docker

```bash
sudo systemctl start docker # 启动服务

# 配置腾讯云的Docker源
sudo vim /etc/docker/daemon.json # 创建配置文件

# 点击键盘I键，切换到I模式，并输入
{
   "registry-mirrors": [
   "https://mirror.ccs.tencentyun.com"
  ]
}

# 按下esc，再输入 ':wq'，这是Vim操作，意思是保存并退出

sudo systemctl restart docker # 重启Docker服务

# 验证源信息，你应该能看到Registry Mirorrs的值是你设置的网址
sudo docker info

# 拉取镜像
sudo docker pull nginx
```


### 杂项

## 引用
- [使用第三方 SSH 终端登录 Linux 实例-操作指南-腾讯云](https://cloud.tencent.com/document/product/1207/44578)
- [使用本地自带 SSH 终端登录 Linux 实例-操作指南-腾讯云](https://cloud.tencent.com/document/product/1207/44643)
- [在 Linux 环境中安装 Docker](https://blog.csdn.net/weixin_48953586/article/details/145597723)
- [Ubuntu 22.04安装Docker](https://blog.csdn.net/weixin_44355653/article/details/140267707)
- [Docker 的官方 GPG 密钥到底是干什么的？](https://blog.csdn.net/qq_36777143/article/details/144872919)
- [云服务器 搭建 Docker-实践教程-腾讯云](https://cloud.tencent.com/document/product/213/46000#1H-kXbk9zoqvzYMVPVsBO)
- [安装 Docker 并配置镜像加速源-实践教程-腾讯云](https://cloud.tencent.com/document/product/1207/45596)
