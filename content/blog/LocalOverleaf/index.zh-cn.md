---
title: 本地 OverLeaf 部署
weight: -5
draft: false
description: Overleaf 本地部署全流程
slug: localoverleaf
tags:
  - overleaf
  - LaTeX
series:
  - 技术流程
series_order: 1
date: 2024-07-12
lastmod: 2024-12-20
authors:
  - Morethan
---

## 背景提要

- 你应该知道如何通过命令行与电脑交互，包括但不限于：Windows如何唤出命令行/终端，正在运行中的命令什么时候结束......

- 懂一点点翻墙的技术，OverLeaf是个国外的软件,与之硬相关的latex项目也是国外的，因此下载相关依赖的时候能够直接接受国外流量会省掉很多麻烦。如果你没有VPN的话就需要为每一个包管理工具指定一个国内源，但有时候国内源更新并不及时。

- 懂一些Vim的基本操作，比如：如何开启插入模式，如何保存退出，不保存退出等


## 部署全流程

### 安装Linux
在 `Windows App Store` 里面直接搜索一个Linux发行版本并下载，笔者选择的是`Kali`。安装完成后可以在开始菜单中直接打开，打开后会跳出命令行窗口，初次打开需要填写需要用户名与密码进行注册。

{{< alert  >}}
此时你的命令行应该有一个 Warning 提示。这是因为你还没有安装 WSL(Windows Subsystem for Linux)；同时，在填写密码的时候你的输入不会显示在命令行，但已经被记录了
{{< /alert >}}
为什么需要一个Linux系统？因为OverLeaf的sharelatex模型需要Linux环境。也正因如此，据说在Linux系统上运行的`OverLeaf`更加流畅。

### 安装WSL
安装WSL2，直接在Windows命令行中运行：

```sh
wsl --install
```

这个程序安装后也可以直接打开，打开后也有一个Warning提示。这时候需要在 `C:\Users\ASUS` 目录下面写入一个text文件，然后重命名为 `.wslconfig`；

写入内容为：

```txt
[experimental] autoMemoryReclaim=gradual # gradual | dropcache | disabled networkingMode=mirrored dnsTunneling=true firewall=true autoProxy=true
```

### 安装Docker
进入 [Docker](https://www.docker.com/) 官网下载Docker，这是sharelatex模型运行的容器。Docker是一个开源的应用容器引擎，其中包括，镜像、容器、仓库，目的就是通过对应用组件的封装、分发、部署、运行等生命周期的管理，使用户的产品及其环境能够做到“一次封装，到处运行”。就像一个集装箱，由程序员开发并封装，用户使用时就直接把整个集装箱搬过去。

Docker安装完成后就可以双击启动放后台了，我们后面通过命令行来操作Docker；

### 拉取镜像

打开 `Kali`，直接运行

``` sh
git clone https://github.com/overleaf/toolkit.git ./overleaf-toolkit
```

然后连续运行：

```sh
cd ./overleaf-toolkit
bin/init
vim ./config/variables.env
```

此时你应该已经进入了一个文档界面，这就是Vim文本编辑器的界面。Vim有很多快捷键，其中按下`"I"`键即可开启插入模式，进行文本编辑，按下`"esc"`即可返回常规模式。在插入模式下输入：`OVERLEAF_SITE_LANGUAGE=zh-CN`

输入完成后按下`"esc"`返回常规模式，直接键入 `:wq` 这是“保存并退出”，如果你不小心输错了可以 `:e!` 放弃所有更改重头再来。这一步是让你的OverLeaf界面显示为中文。

当你成功保存并退出，回到熟悉的`Kali`命令行界面后运行 `bin/up` 此时正在拉取sharelatex镜像以及相关的网络工具。这时会有大量的数据传输，要保证网络通畅（梯子要稳！）

### 配置用户
当上一个命令成功结束之后，运行 `bin/start` ；此时你打开Docker点进sharelatex，你应该可以看到代码“爆闪”，如果没有红色的消息，那应该是正常运行了。

这时打开浏览器访问网址 `http://localhost/launchpad` 

按照提示注册Administrator Account之后，就会跳转到 `http://localhost/project` ；这时基本的OverLeaf网页已经可以显示了。

{{< alert icon="pencil" cardColor="#1E3A8A" textColor="#E0E7FF" >}}
但现在你丢一个文件进去编译多半是会报错的 `ᕕ( ᐛ )ᕗ` ；因为此时 sharelatex 里面的宏包严重不足，不是红包「手动狗头」
{{< /alert >}}
### 安装扩展包

打开 `Kali` 进入对应目录运行 `bin/shell` 然后逐条执行：

```sh
cd /usr/local/texlive

# 下载并运行升级脚本
wget http://mirror.ctan.org/systems/texlive/tlnet/update-tlmgr-latest.sh
sh update-tlmgr-latest.sh -- --upgrade

# 更换texlive的下载源
tlmgr option repository https://mirrors.sustech.edu.cn/CTAN/systems/texlive/tlnet/

# 升级tlmgr
tlmgr update --self --all

# 安装完整版texlive（时间比较长，不要让shell断开）
tlmgr install scheme-full

# 退出sharelatex的命令行界面
exit

# 重启sharelatex容器
docker restart sharelatex
```

重启后再次进入 `shell`，运行：

```sh
apt update

# 安装字体
apt install --no-install-recommends ttf-mscorefonts-installe fonts-noto texlive-fonts-recommended tex-gyre fonts-wqy-microhei fonts-wqy-zenhei fonts-noto-cjk fonts-noto-cjk-extra fonts-noto-color-emoji fonts-noto-extra fonts-noto-ui-core fonts-noto-ui-extra fonts-noto-unhinted fonts-texgyre

# 安装pygments
apt install python3-pygments

# 安装beamer之类的
apt install texlive-latex-recommended
apt install texlive-latex-extra

# 安装英文字体
echo "yes" | apt install -y --reinstall ttf-mscorefonts-installer

# 安装中文字体
apt install -y latex-cjk-all texlive-lang-chinese texlive-lang-english
cp fonts/* /usr/share/fonts/zh-cn/
cd /usr/share/fonts
fc-cache -fv # 更新字体缓存
fc-list :lang=zh-cn
fc-match Arial
```

最后在`shell`目录里面运行：
```sh
vim /usr/local/texlive/2023/texmf.cnf
```
进入配置文件，在最底下加入一句 `shell_escape = t`

{{< alert icon="pencil" cardColor="#1E3A8A" textColor="#E0E7FF" >}}
我也不知道这有什么用，属于是前辈传承了🤔
{{< /alert >}}
注意，如果Texlive(扩展包的官名)版本不同的话，目录地址也会有所变化，因此需要根据实际的地址来填写，例如将`2023`改成`2024`。

{{< alert icon="pencil" cardColor="#1E3A8A" textColor="#E0E7FF" >}}
在Linux命令行中可以用 `ls -l` 来查看当前目录下所有的文件
{{< /alert >}}
## 部署成功
现在你就可以愉快地使用本地版OverLeaf了，没有编译超时的困扰~

如果非常巧合，你也是个CQUer，这里附赠一份重庆大学的毕业论文模板，炒鸡的亲民哦：[CQUThesis](https://github.com/nanmu42/CQUThesis)
