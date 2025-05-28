---
title: 云服务搭建
weight: -85
draft: true
description: 详细记录了从零开始的云服务平台搭建过程
slug: cloud-server-build
tags:
  - 云服务
series:
  - 技术杂项
series_order: 7
date: 2025-03-06
lastmod: 2025-03-16
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

#### 大体思路

如果你是一个真正意义上的"独立"开发者，没有任何人需要与你进行协同开发，那么你完全不需要一个云服务器，你只需要一个 Docker 就行。

安装 DockerDesktop 之后，你需要根据你已有的后端服务代码进行**容器编排**

这一步主要是：

1. 建立 API 接口：编写主操作逻辑，这里的主操作逻辑可以使用任何你喜欢的编程语言来编写
2. 创建必要的环境文件：例如 `.env` `pyproject.toml` 等环境文件
3. 创建 DockerFile：其实就是一个命令行指令集，用来初始化你的容器
4. 创建 `docker-compose.yml` 文件：在这个文件中你需要去编排你的 app 用到的镜像，指定内部网络、端口、挂载卷等等必要设置

然后一个命令：`docker-compose up --build`

Docker 初始化完成之后，你就可以直接通过 `localhost` 这个域名来访问你的服务了，就和使用云服务器的效果一样。

#### Docker路径问题

在编排 Docker 容器的时候，有几个必要的**工作路径参数**需要了解，否则很容易产生"文件无法找到"的错误😢

1. `build.congtext`：这个参数存在于 `docker-compose.yml` 中，就是所谓的"构建上下文"，指向的是你本地的一个真实的目录
2. `WORKDIR`：这个参数存在于 `Dockerfile` 文件中，用来指定"工作路径"；后续的 RUN、COPY 等指令如果你使用了**相对路径**则都会在这个路径中执行，但是不影响绝对路径参数

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

网上有很多教程，这里就不赘述了，找一个你信得过的域名注册厂商就可以了。

如果你不知道域名是什么，这里有我的一个通俗解释：你家的小区的正式地址你记得全吗？但你总是记得你家小区的名字吧？域名就相当于你家小区的名字。作用就俩字：好记

当然对于大公司来说还有：气派😄

### 设置 DNS 解析

DNS 解析服务一般是在域名注册厂商的服务，你购买了域名之后一般就会赠送免费的 DNS 解析服务。虽然是低配版，但好歹能用👌

DNS 解析是啥？域名相当于你家小区的名字，但外地人根本就不知道。DNS 就相当于一个翻译，把你家小区的名字翻译成全国通用的形式，也就是你家小区的正式地址：xx 省 xx 事 xx 街道......

这样即便是外地人也能来你家串门了，不会走错地方。

### 购买云服务器

同样是从云服务器供应商那里买就行了，这套服务体系已经非常成熟了。没什么好说的了😅

### 配置服务器

我这里使用的服务器操作系统是 Ubuntu Server 24.04 LTS 64bit，后续操作命令也是基于这个系统

#### 文件传输

为了从本地传输文件到服务器上，我选择使用一些更为现代化的终端，比如：[Tabby](https://tabby.sh/), [WindTerm](https://github.com/kingToolbox/WindTerm) 和 [Warp](https://www.warp.dev/) 等等

我随便选择了 Tabby 这个终端，它可以更加方便地进行远程连接，并且内嵌了 SFTP 方便传输文件，而且如果你有精力来折腾终端界面，那么 Tabby 还是能够非常美观的😄

当然，传统一些的方案可以选择使用 [FileZilla](https://filezilla-project.org/download.php)；而且不怕麻烦的话直接用终端命令也是可以的：使用 `rsync` 或者 `scp` 命令就行


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

echo   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://mirrors.cloud.tencent.com/docker-ce/linux/ubuntu/ \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" |   sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

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

sudo systemctl stop docker # 停止Docker服务

# 验证源信息，你应该能看到Registry Mirorrs的值是你设置的网址
sudo docker info

# 拉取镜像
sudo docker pull nginx

# 如果你想使用docker-compose
# 在Ubuntu上请使用：docker compose
sudo docker compose up --build

# 启动服务但不重新构建
sudo docker compose up -d

# 停止所有服务
sudo docker compose down
```


{{< alert icon="triangle-exclamation" cardColor="#ffcc00" textColor="#333333" iconColor="#8B6914" >}}
在使用 `docker compose` 命令之前，请按照官方教程自行创建 `docker-compose.yml` 和 `Dockerfile`
{{< /alert >}}

#### 配置pip/poetry

很奇怪，腾讯云在系统层面默认安装了 Python，但是没有安装 pip 🤔所以需要手动安装一下，在安装前请先确认一下 Python 是否安装


{{< alert icon="triangle-exclamation" cardColor="#ffcc00" textColor="#333333" iconColor="#8B6914" >}}
如果你使用的不是系统层面的 pip，而是 Docker 容器内部的 pip，那么对外部系统层面的 pip 进行换源操作没有效果
{{< /alert >}}

系统层面更换 pip 源：

```bash
# 查询Python版本
python3 -V

# 查看python3的路径
which python3

# 确认安装了python并且没有安装pip
sudo apt install python3-pip

# 如果使用的是腾讯的服务器，可以使用内网域名，速度更快
pip config set global.index-url https://mirrors.tencentyun.com/pypi/simple

# 如果不是，那就使用外网域名
pip config set global.index-url https://mirrors.cloud.tencent.com/pypi/simple
```

Docker 容器内部更换 pip 源：直接在你的 `Dockerfile` 文件里面写入如下命令

```bash
# 配置pip源
# 使用腾讯云镜像源，注意这里是内网域名
RUN pip config set global.index-url https://mirrors.tencentyun.com/pypi/simple \
    && pip config set global.trusted-host mirrors.tencentyun.com
```

如果你在系统层面使用 poetry：

```bash
# 使用在线脚本安装poetry
curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python

# 使用apt安装poetry
sudo apt install python3-poetry

# 使用腾讯云镜像源，注意这里是内网域名
sudo poetry config repositories.tencentyun https://mirrors.tencentyun.com/pypi/simple
```


{{< alert icon="pencil" cardColor="#1E3A8A" textColor="#E0E7FF" >}}
如果你的系统没有默认安装 pip，那么不建议先安装 pip 再安装
{{< /alert >}}

#### SSL/TLS

SSL/TLS 是一套加密传输协议，经过密码学专家的精心设计。我们现在广泛使用的**超文本安全传输协议**(HTTPS)就是在明文传输协议 HTTP 的基础上使用 SSL/TLS 加密后得到的。

HTTPS核心的一个部分是**数据传输之前**的握手，握手过程中确定了数据加密的密码。在握手过程中，网站会向浏览器发送SSL证书，SSL证书和我们日常用的身份证类似，是一个支持HTTPS网站的身份证明，SSL证书里面包含了网站的域名，证书有效期，证书的颁发机构以及用于加密传输密码的公钥等信息。浏览器在生成密码之前需要先核对当前访问的域名与证书上绑定的域名是否一致，同时还要对**证书的颁发机构**(CA)进行验证，如果验证失败浏览器会给出证书错误的提示。

一般 SSL 证书需要向 CA 机构购买，但是也有一些 CA 机构提供了免费的 SSL 证书服务，比如：[Let’s Encrypt](https://letsencrypt.org/)

并且经过长期发展，Let’s Encrypt 颁发的证书已经可以通过 Certbot 来进行自动化处理了，证书到期了还能自动再次申请。当然如此的方便自然是要付出一些代价的😢：你需要配置 certbot，不得不说还是挺麻烦的

这里我选择直接在 Docker 容器内部部署一个 certbot 来进行 SSL 证书的申请，也就是说可以直接修改 `docker-compose.yml` 中的内容，拉取官方的 certbot 镜像。

```bash
# 先关停所有的Docker服务
sudo docker compose down

# 启动所有服务
sudo docker compose up -d

# 启动某部分服务
sudo docker compose up nginx -d

# 查看特定镜像的日志
sudo docker logs server-app-1

# 查看特定服务的日志
sudo docker compose logs certbot-init

# 使用递归删除来清除缓存
rm -rf ./certbot/conf/*

# 进入一个镜像内部
sudo docker exec -it server-app-1 /bin/sh

# 删除某个镜像
sudo docker rmi -f server-frontend

# 重新启动某项服务
sudo docker compose restart app

# 清理npm的缓存
npm cache clean --force

# 激活python虚拟环境
source your_venv_name/bin/activate
```

### 备案

使用企业主体进行备案没有什么难度，因此不过多赘述，这里主要讲一下**个人主体**备案。主要参考：[腾讯云服务器备案全流程 40天备案的血与泪-博客园](https://www.cnblogs.com/yyzwz/p/13393223.html)

{{< mermaid >}}
graph LR;
p1(服务器平台实名认证)-->p2(实名认证购买域名)-->p3(申请备案)-->p4(平台审核)-->p5(管局审核)-->p6(公安备案)
{{< /mermaid >}}

如果你是腾讯云用户，那么就可以使用**腾讯云网站备案小程序**；提交申请之后平台会先进行审核，一般 8 个小时左右；然后平台会提交管局审核，一般 7 个工作日左右；在管局审核通过之后需要在 30 个工作日内进行公安备案，

### 杂项

这里主要写一下整个操作过程中可能遇到的问题。

#### certbot访问被拒绝了？

首先，当然是检查网络原因，检查一下服务器的安全组放行的端口有没有 443 和 80 端口，检查一下服务器操作系统的防火墙有没有拦截端口访问。

```bash
# Ubuntu默认安装uwf防火墙
# 查看uwf状态，inactive状态是未启动，也是理想状态
sudo ufw status
```

如果你是国内用户的话，还有一个原因：ICP 备案没有通过，certbot 也是无法访问你的服务器的😢

## 引用

- [使用第三方 SSH 终端登录 Linux 实例-操作指南-腾讯云](https://cloud.tencent.com/document/product/1207/44578)
- [使用本地自带 SSH 终端登录 Linux 实例-操作指南-腾讯云](https://cloud.tencent.com/document/product/1207/44643)
- [在 Linux 环境中安装 Docker](https://blog.csdn.net/weixin_48953586/article/details/145597723)
- [Ubuntu 22.04安装Docker](https://blog.csdn.net/weixin_44355653/article/details/140267707)
- [Docker 的官方 GPG 密钥到底是干什么的？](https://blog.csdn.net/qq_36777143/article/details/144872919)
- [云服务器 搭建 Docker-实践教程-腾讯云](https://cloud.tencent.com/document/product/213/46000#1H-kXbk9zoqvzYMVPVsBO)
- [安装 Docker 并配置镜像加速源-实践教程-腾讯云](https://cloud.tencent.com/document/product/1207/45596)
- [Debian 12 / Ubuntu 24.04 安装 Docker 以及 Docker Compose 教程-烧饼博客](https://u.sb/debian-install-docker/)
- [手把手教你在Linux系统下进行Python pip换源操作-腾讯云](https://cloud.tencent.com/developer/article/1635489)
- [pip源配置-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1601851)
- [poetry如何更换国内源-数据科学SourceResearch](https://www.resourch.com/archives/66.html)
- [Linux停止 Docker 容器：单个、多个或全部](https://cn.linux-console.net/?p=20047)
- [docker日志在哪看?怎么在Linux服务器中查看日志-CSDN博客](https://blog.csdn.net/qq_35393472/article/details/136719510)
- [Linux 防火墙关闭后仍无法访问 Web 的问题排查与解决](https://cloud.baidu.com/article/2693474)
- [关于ubuntu控制台开放了所有端口,但是外部主机依然无法访问对应端口服务问题_ubuntu端口开放后不通-CSDN博客](https://blog.csdn.net/weixin_51125521/article/details/132273384?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-132273384-blog-106063657.235%5Ev43%5Econtrol&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-132273384-blog-106063657.235%5Ev43%5Econtrol&utm_relevant_index=2)
- [三分钟带你详解SSL认证与加密技术-CSDN博客](https://blog.csdn.net/qq_16481211/article/details/119860572)
- [一篇文章让你彻底弄懂SSL/TLS协议 - 知乎](https://zhuanlan.zhihu.com/p/133375078)
- [HTTPS 与 SSL 证书概要 | 菜鸟教程](https://www.runoob.com/w3cnote/https-ssl-intro.html)
- [常用的四种免费证书申请方式-CSDN博客](https://blog.csdn.net/weixin_45444133/article/details/120900424)
- [使用docker部署nginx并配置https - TandK - 博客园](https://www.cnblogs.com/tandk-blog/p/15449873.html)
- [使用 Docker + Nginx + Certbot 实现自动化管理 SSL 证书-CSDN博客](https://blog.csdn.net/hjj19930729/article/details/145384756)
- [The Certificate Authority failed to download the temporary challenge files created by Certbot -- Connection refused - Help - Let's Encrypt Community Support](https://community.letsencrypt.org/t/the-certificate-authority-failed-to-download-the-temporary-challenge-files-created-by-certbot-connection-refused/159426/26)
- [ICP 备案 首次备案_腾讯云](https://cloud.tencent.com/document/product/243/97668)
- [ICP 备案 各省管局要求_腾讯云](https://cloud.tencent.com/document/product/243/3474)
- [腾讯云服务器备案全流程 40天备案的血与泪 - 郑为中 - 博客园](https://www.cnblogs.com/yyzwz/p/13393223.html)
- [ICP 备案 公安备案流程_腾讯云](https://cloud.tencent.com/document/product/243/19142)
- [ICP 备案 视频核验_腾讯云](https://cloud.tencent.com/document/product/243/34945)
- [npm缓存深度解析：理解、使用与清除指南_npm 清除缓存-CSDN博客](https://blog.csdn.net/2401_85762266/article/details/140127881)
- [用Docker部署一个Web应用 - 知乎](https://zhuanlan.zhihu.com/p/26418829)
- [HTTPS请求为什么会降级为HTTP-CSDN博客](https://blog.csdn.net/weixin_43947690/article/details/88295278)
- [https 被redirect成了http_风控-CSDN博客](https://blog.csdn.net/zhuye1992/article/details/80496151?spm=1001.2101.3001.6650.4&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-4-80496151-blog-88295278.235%5Ev43%5Econtrol&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-4-80496151-blog-88295278.235%5Ev43%5Econtrol&utm_relevant_index=9)
- [如何填写公安联网备案公安联网备案信息指南_备案(Filing Service)-阿里云帮助中心](https://help.aliyun.com/zh/icp-filing/basic-icp-service/the-public-security-network-for-the-record-information-fill-in-the-guide#:~:text=%E8%8B%A5%E6%82%A8%E5%8A%9E%E7%90%86%E5%85%AC%E5%AE%89%E8%81%94%E7%BD%91%E5%A4%87%E6%A1%88%E7%9A%84%E5%9F%9F%E5%90%8D%E6%98%AF%E9%80%9A%E8%BF%87%E9%98%BF%E9%87%8C%E4%BA%91%E6%B3%A8%E5%86%8C%EF%BC%8C%E5%8D%95%E5%87%BB%20%E6%9F%A5%E8%AF%A2%E7%BD%91%E7%BB%9C%E6%B3%A8%E5%86%8C%E6%9C%8D%E5%8A%A1%E5%95%86%E3%80%82%20%E6%A0%B9%E6%8D%AE%E5%A6%82%E4%B8%8B%E9%98%BF%E9%87%8C%E4%BA%91%E4%BF%A1%E6%81%AF%E5%A1%AB%E5%86%99%E3%80%82%20%E8%AF%B7%E6%A0%B9%E6%8D%AE%E7%BD%91%E7%AB%99%E5%AE%9E%E9%99%85%E5%BC%80%E5%B1%95%E7%9A%84%E4%B8%9A%E5%8A%A1%E9%A1%B9%E7%9C%9F%E5%AE%9E%E9%80%89%E6%8B%A9%EF%BC%8C%E4%BE%8B%E5%A6%82%EF%BC%9A%E7%BD%91%E7%BB%9C%E5%9F%BA%E7%A1%80%E7%B1%BB%20A%E3%80%81%E7%BD%91%E7%BB%9C%E9%94%80%E5%94%AE%E7%B1%BB%20B%E3%80%81%E7%94%9F%E6%B4%BB%E6%9C%8D%E5%8A%A1%E7%B1%BB%20C%20%E5%92%8C%E7%A4%BE%E4%BA%A4%E6%96%87%E5%A8%B1%E7%B1%BB,D%20%E7%AD%89%E3%80%82%20%E5%A6%82%E6%9E%9C%E4%B8%8D%E6%B6%89%E5%8F%8A%EF%BC%8C%E5%B0%86%20%E6%98%AF%E5%90%A6%E6%8F%90%E4%BE%9B%E4%BA%92%E8%81%94%E7%BD%91%E4%BA%A4%E4%BA%92%E6%9C%8D%E5%8A%A1%20%E9%80%89%E6%8B%A9%E4%B8%BA%20%E5%90%A6%E3%80%82%20%E5%A6%82%E6%9E%9C%E6%B6%89%E5%8F%8A%EF%BC%8C%E6%A0%B9%E6%8D%AE%E7%BD%91%E7%AB%99%E5%AE%9E%E9%99%85%E5%BC%80%E5%B1%95%E7%9A%84%E4%B8%9A%E5%8A%A1%E9%A1%B9%E7%9C%9F%E5%AE%9E%E9%80%89%E6%8B%A9%E3%80%82%20%E6%98%AF%E5%90%A6%E6%8F%90%E4%BE%9B%E6%B6%89%E5%8F%8A%E7%AE%A1%E5%88%B6%E7%89%A9%E5%93%81%E4%BF%A1%E6%81%AF%E5%8F%91%E5%B8%83%E6%9C%8D%E5%8A%A1%EF%BC%9A%E8%AF%B7%E6%8C%89%E7%85%A7%E7%BD%91%E7%AB%99%E6%B6%89%E5%8F%8A%E8%8C%83%E5%9B%B4%E5%A6%82%E5%AE%9E%E5%8B%BE%E9%80%89%E3%80%82)
