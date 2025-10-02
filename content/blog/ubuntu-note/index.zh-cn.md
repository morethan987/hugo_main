---
title: Ubuntu折腾札记
weight: -90
draft: false
description: 记录折腾Ubuntu系统的过程以备不时之需
slug: ubuntu-note
language: zh-cn
tags:
  - Ubuntu
  - Linux
series:
  - 技术杂项
series_order: 8
date: 2025-05-01
lastmod: 2025-10-01
authors:
  - Morethan
---

{{< lead >}} 总结记录一下折腾Ubuntu系统的过程，以备不时之需 {{< /lead >}}

## 前言

Ubuntu 作为一个流行的 Linux 发行版本，对比其他 Linux 发行版，有较好的生态支持。最主要的体现就是：当你遇到问题的时候 Ubuntu 能够搜到的教程更多。

这篇文章主要针对带有 GUI 的个人版 Ubuntu 系统，服务器专用的 Ubuntu 系统操作取决于你的实际业务，[云服务器搭建]({{< ref "/blog/cloud-server-build/" >}})这篇文章可以作为一个参考。

## Ubuntu安装

非常久远之前就安装好了，故不做记录了。如有需要，请自行搜索“移动硬盘安装 Ubuntu”等关键词。这里放一个较新的链接：[移动硬盘安装Ubuntu](https://blog.csdn.net/qq_52034548/article/details/131581118)

安装过程已经不可考究了，现状就是我将 Ubuntu 系统安装在一个移动硬盘中，随插随用。

使用的时候就可以在开机之前把硬盘插上，点击电源按钮之后快速点击某个按键进入 BIOS 引导界面，把优先级提升到最高然后保存退出就可以正常进入 Ubuntu 系统了。

不想使用的时候直接不插上硬盘就可以了，正常开机，不需要任何的操作就可以直接进入 Windows 系统，非常地方便。

## 界面美化

我个人比较喜欢苹果风格的界面，因此特地找了一款苹果风格的主题：[WhiteSur](https://github.com/vinceliuice/WhiteSur-gtk-theme)

安装过程非常简单：

```bash
git clone https://github.com/vinceliuice/WhiteSur-gtk-theme.git --depth=1

cd WhiteSur-gtk-theme

./install.sh # 运行安装脚本
```

具体的配置操作请见 Github 官网上的说明

至于如何升级，官方没有说明，估计是默认用户都知道：

```bash
git pull # 拉取最新的代码

./install.sh # 重新运行安装脚本即可
```

## 文本编辑

此处的文本编辑特指的是在命令行中的文本编辑，在专门的文本编辑器中的文本编辑实在是太直观简单了，根本不用学。而且绝大多数情况下，我们也不用在命令行中进行文本编辑。

然而有些时候你不得不在命令行中编辑文本：服务器上连 GUI 都没有😢虽然可以通过 VS Code 的插件在服务器上运行 GUI 编辑界面，但是当你只想改一个配置文件，启动如此"重量级"的编辑器实属牛刀杀鸡了🤔

如果你只是想创建一个文件，然后用别的编辑器进行编辑，那么简单一行命令即可：

```bash
# 碰一下就会出现😉
touch file_name.md
```

### nano

这个命令行编辑器是 Ubuntu 系统自带的命令行文本编辑器，不用专门下载。

想要编辑文件的话也非常简单：

```bash
nano file_name
```

进入 nano 之后就有几行提示，标出了相关的快捷键，按照提示进行操作就行了，属于是非常简单的命令行文本编辑器了。

### vim

我很早就听说过这个"传奇上古"编辑器的大名，并且听说了它所拥有的各种各样的优点：手指不离开键盘有助于思维输出、习惯之后就戒不掉了、简洁轻量化能够高度自定义、牛逼的程序员都用vim……

然而我几经尝试后得出结论：vim 很好，但我真的很懒😢我真的不想花几个星期去和各种各样的组合键进行磨合

这里我只记录一些最最最常用的操作，适合偶尔使用 vim 进行轻量编辑的同志，看完下面这些操作你就可以对外宣称你会用 vim 了😋

```bash
i # 进入插入模式

<ESC> # 返回正常模式

: # 进入命令模式

:q! # 退出不保存

:wq # 退出保存

y6w # 复制6个词

p # 粘贴

dd # 删除行

dw # 删除一个词
```


{{< alert icon="pencil" cardColor="#1E3A8A" textColor="#E0E7FF" >}}
网上有很多非常详细的 vim 教程，如果想仔细学 vim 的话可以去看看👀看得出来，那些教程真的很想让我成为一个 vim 大师😔但我是真的懒😔
{{< /alert >}}

### neovim

出于各种各样不可抗的原因，偶尔不得不使用终端 IDE 来进行项目开发😢但是传统的 vim 实在是太"硬核"了，偶然听说 neovim 非常好用故尝试一下。

**Neovim** 是 Vim 的现代重构版本，全称 *Neo (New) Vim*，目标是为老牌编辑器 Vim 带来现代化的特性，包括：

- 更好的插件系统（异步、Lua 支持）
- 更强的语言服务集成（LSP）
- 更友好的可配置性（用 Lua 而不是 VimScript）   
- 更适合与 IDE 特性集成（调试、补全、跳转等）

#### 目录结构

目录 | 位置 | 作用
----- | ----- | -----
`~/.config/nvim/` | 主配置目录 | 存放 `init.lua/init.vim` 和自定义插件配置
`~/.local/share/nvim/` | 数据目录 | 插件仓库、自动下载的插件等内容（由插件管理器创建）
`~/.cache/nvim/` | 缓存目录 | 保存缓存、LSP 日志、telescope 索引、treesitter 缓存等
`~/.local/state/nvim/` | 状态目录 | 运行时的临时状态（崩溃记录、运行中数据）
`~/.config/nvim/lazy/` | 插件配置 | 插件管理器生成的插件加载信息

主配置目录结构：

```text
~/.config/nvim/
├── init.lua                 # 入口文件，类似 VS Code 的 settings.json
├── lua/                     # 存放你所有 Lua 配置模块
│   ├── core/                # 通用设置、快捷键、自定义函数等
│   ├── plugins/             # 插件配置文件（每个插件一个文件）
│   └── lazy.lua             # lazy.nvim 插件加载器初始化文件
├── after/                   # 在插件加载之后执行的额外设置
│   └── plugin/              # 如文件类型自动命令
└── plugin/                  # 自动加载的 Lua/Vim 脚本
```

#### 配置方案

现在许多非常酷炫的终端 IDE 的背后其实就是 neovim，而这些 IDE 本身其实是 neovim 的一套成品配置方案，是 neovim 玩家自己折腾出来的成果。比较出名的有：[LazyVim](https://github.com/LazyVim/LazyVim) 和 [NvChad](https://github.com/NvChad/NvChad)

这些第三方的配置方案一般都有比较完善的说明文档，按照官方说明直接安装即可。

## Fcitx 5

主要参考：[在 Ubuntu 安装配置 Fcitx 5 中文输入法](https://muzing.top/posts/3fc249cf/)；之前有考虑过使用搜狗输入法，但是看到官方给的安装流程就头大，而且也需要安装 Fcitx 5，所以不如直接用 Fcitx5 了

其实一开始我是拒绝 Fcitx 的🥲因为它的界面实在是"朴素"得令人难以接受，相信上面那篇博客的作者应该和我深有同感👆

## 使用Windows上的字体

思路：将 Windows 中的字体文件拷贝到 Ubuntu 专用的字体文件夹中，然后给予适当的权限，然后刷新 Ubuntu 的字体缓存，加载新的字体。

```bash
# Windows上的字体文件夹：C:/Windows/Fonts
sudo cp /mnt/C/Windows/Fonts/LXGWWenKai-Regular.ttf /usr/share/fonts/custom/LXGWWenKai-Regular.ttf

# 提高权限
sudo chmod u+rwx /usr/share/fonts/custom/*

# 进入字体文件夹
cd /usr/share/fonts/custom/

# 建立字体缓存
sudo mkfontscale

# 刷新缓存
sudo fc-cache -fv
```

当然，你也可以直接从网上下一个新的 `.ttf` 文件然后再复制到指定文件夹中。而且如果你使用的是带有 GUI 界面的 Ubuntu 系统，下载文件之后你可以直接双击字体文件来安装🥰


{{< alert icon="triangle-exclamation" cardColor="#ffcc00" textColor="#333333" iconColor="#8B6914" >}}
字体可能会重复安装，系统不会检查某个字体是否重复安装；如果安装重复了请自行去相关文件夹中查找并手动删除🥲Ubuntu 中字体工具可以查看所有字体信息
{{< /alert >}}

## 配置VPN

本人一般使用 clash+airport 来获取外部网络资源

首先当然是按照 airport 的教程说明，下载 clash 并获取配置文件，然后一般会通过一个命令来启动 VPN：

```bash
./clash -d .
```

然而，这条命令会占用一个终端，并且每次开机之后都要手动输入一遍，非常不方便。

于是可以把这个命令封装为一个系统服务，可以使用 `systemctl` 来进行控制管理，也可以直接设置为开机自启动。

样例配置文件，需要写入 `/etc/systemd/system/clash.service` 中：

```service
[Unit]
Description=Clash Service
After=network.target

[Service]
Type=simple
User=morethan
ExecStart=/home/morethan/Program/clash/clash -d /home/morethan/Program/clash
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

设置为开机自启动：

```bash
sudo systemctl daemon-reexec && sudo systemctl enable clash.service
```

使用 `systemctl` 进行控制：

```bash
# 立即启动
sudo systemctl start clean-up.service

# 立即停止
sudo systemctl start clean-up.service
```

当然，也可以在：`系统设置 --> 网络 --> 代理` 中手动进行设置

## ssh反向代理

很多时候我们都需要使用远程服务器来进行代码开发工作，但是国内远程服务器的网络有非常难以处理的特性：连不上外网，甚至根本就无法访问国内网络。

之所以说这是特性，主要原因是我还真就没见过能够同时满足上述两个网络需求的服务器😭我的一份干了好几个月的代码就因为网络原因丢失了，现在想起来都十分痛心😭

于是我痛定思痛，决定彻底解决这个棘手的问题：以后只要我能够连接上这个远程服务器，我就一定要让其能够流畅地访问外部网络。俗话说得好，磨刀不误砍柴工，网络问题真的不是一个可以忽视的问题。

下面这些内容主要来自于这篇博客文章：[SSH反向隧道使远程计算机连接互联网教程](https://www.bonanzhu.com/blog/2024/ssh-reverse-proxy-usage-chinese/)

核心命令如下：

```
# 开启一个终端并启动反向代理
ssh -R 7890:localhost:7890 username@remote_host
```

这个命令的含义是：在远程电脑上打开7890端口，并将所有发往这个端口的数据转发到本地电脑的7890端口（即你的代理服务器）。这个命令执行之后就会开启一个远程服务器的终端，在这个终端中可以立即测试代理是否成功：

```bash
https_proxy=http://localhost:7890 curl https://www.baidu.com
```

如果你的本地电脑上的 7890 端口恰好运行了一个 VPN，那么你还能够尝试访问外网的资源：

```bash
https_proxy=http://localhost:7890 curl https://www.google.com
```

如果一切正常的话你现在应该能够获取到外部网络的资源了😄

但是上面这种方法我觉得不太优雅：每次想要给一个服务器进行反向代理就需要运行这个命令，并且还会占用一个终端窗口

因此我将这个功能封装为了一个系统服务模板，能够使用类似下面这个命令来进行控制：

```bash
# 启动反向代理
sudo systemctl start ssh-reverse-proxy@A6000.service

# 查看代理状态
sudo systemctl status ssh-reverse-proxy@A6000.service

# 停止反向代理
sudo systemctl stop ssh-reverse-proxy@A6000.service
```

其中@后面的是一个可变参数，你只需要填入你想访问的目标服务器的别名即可。当然，你需要在 `~/.ssh/config` 中写入你的服务器别名。当然为了方便起见，还需要配置免密登陆，详见下一节的内容：[ssh免密登录]({{< relref "#ssh免密登录" >}})

当然还可以将上面这个冗长的命令封装为一个 alias，具体 alias 的命名就看个人喜好了，简洁易懂就行。

下面是配置系统级服务的命令操作：

```bash
sudo vim /etc/systemd/system/ssh-reverse-proxy@.service
```

写入下面这些内容，当然有一些必要内容是要修改的：

```service
[Unit]
# Description中的'%i'会被您传入的参数（SSH别名）替换
Description=SSH Reverse Proxy to %i
After=network-online.target
Wants=network-online.target

[Service]
# [务必修改] 替换为您的本地用户名，以便服务能找到您的SSH密钥
User=your_local_user

# ExecStart中的'%i'也会被替换为SSH别名
# 这就是实现参数化的核心
ExecStart=/usr/bin/ssh -NT \
    -o ServerAliveInterval=60 \
    -o ExitOnForwardFailure=yes \
    -R 7890:localhost:7890 %i

# 自动重启策略
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

然后重载一下系统服务，就可以使用上文提及到的 `systemctl` 来控制这个反向代理服务了😄

另外如果想要使用 git 的话，就需要利用 `netcat` 将 ssh 的流量转接到 https。说着比较抽象，但是实际上具体需要做的操作并不多，只需要在 `~/.ssh/config` 中写入下面的内容：

```
Host github.com gitlab.com
  # 匹配多个主机，用空格隔开

  # [核心配置]
  # 使用 netcat (nc) 通过 HTTP 代理连接
  # -x connect_address:port 指定HTTP代理地址
  # -X 5 指定代理协议为 SOCKS5 (如果代理支持)
  #  - or -
  # -X connect 指定使用 HTTP CONNECT (更通用)
  ProxyCommand nc -X connect -x localhost:7890 %h %p
  
  User git
```

然后按照 [GitHub-SSH]({{< relref "#github-ssh" >}}) 中的内容配置 GitHub 免密推送即可

## ssh免密登录

对于没有本地 GPU 的学生党来说，直接在服务器上编辑并运行文件是非常方便的，而 VSCode 插件 `remote-ssh` 也能够在服务器上直接呈现一个 VSCode 的界面。为了省去不断输入密码的痛苦，可以进行如下操作实现免密登录😄

本地机为 Windows，一般来说在 `C:\Users\<User_name>\.ssh` 文件夹中会存放 `ssh` 相关的配置文件，实现免密登录就只需要操作这里面的文件即可。

1. 配置 `config` 文件，样例配置如下：

```txt
# 把<User_name>替换为你的本地机用户名
Host 3090
    HostName xx.xxx.xx.xx
    User morethan
    IdentityFile "C:\Users\<User_name>\.ssh\id_rsa"
```

2. 生成验证文件 `id_rsa.pub`

3. 本地创建 `authorized_keys` 文件，并将 `id_rsa.pub` 中的内容写入这个文件

4. 在服务器中默认目录下创建 `.ssh` 文件夹，然后将 `authorized_keys` 文件从本地拷贝进去即可

然后用 VSCode 登录服务器就发现已经完成了免密登录


{{< alert icon="pencil" cardColor="#1E3A8A" textColor="#E0E7FF" >}}
上述繁琐的步骤只需要初次执行一次就行，当需要给下一台服务器配置免密登录时，只需要改 config 文件然后把 authorized_keys 文件拷贝到服务器就完成了，不需要改写任何文件😄
{{< /alert >}}

## 挂载硬盘

由于我的 Ubuntu 系统安装在移动硬盘上，因此这里主要解决的问题是：如何在 Ubuntu 上访问 Winodws 中的盘区。因此不涉及对于分区的具体处理，如果需要对分区进行格式化等等操作，详见：[如何在Ubuntu系统中进行磁盘的分区与挂载](https://cloud.tencent.com/developer/article/2456171)

```bash
# 查看磁盘及其分区，sudo提权不可省略
sudo fdisk -l

# 创建挂载位点，其实就是建一个文件夹
# mnt中的子文件夹之所以取名为E是，因为这里准备挂载E盘
sudo mkdir /mnt/E

# 直接挂载新分区
sudo mount /dev/vdb /mnt/E

# 设置开机自动挂载
# 查看分区的UUID
sudo blkid

# 编辑特定文件
vim /etc/fstab

# 在文件末尾追加
UUID=xxxxxxxx /mnt/E ntfs defaults 0 2
```

- 上面的 UUID 请替换为 `blkid` 命令获取的内容，`ntfs` 处也替换为相应的文件系统类型（常见的如 `ntfs` `ext4`）
- `defaults`：这是一个组合选项，包含一组默认挂载选项，如rw（读写）、relatime（减少inode访问时间更新次数）等
- 0和2：这些值分别控制是否需要备份和文件系统检查顺序。通常第一个值为 0（不备份），第二个值为1或2（1用于根文件系统，其他文件系统用2）

**测试方法**：

```bash
# 运行后如果没有报错则说明配置正确
sudo mount -a
```

## 创建快捷方式

常见的一个操作：我想在桌面放一个常用文件夹的快速链接，方便快速进入对应文件夹

```bash
# 在Desktop文件夹中放置一个指向/target/dir文件夹的链接
# 请替换为你的目标文件夹
ln -s /target/dir ~/桌面

# 测试，如果能cd进去那么就没问题
cd ~/桌面/dir
```

## 系统信息

在没有 UI 界面的时候，就需要用命令行来查看系统的相关信息

```bash
# 系统信息概要
hostnamectl

# CPU详细信息
lscpu

# 查看 CPU 使用情况
top

# 分区使用情况
df -h
```

## 配置Git

Linux 的一大特色就是极致的简洁，因此使用命令行来操纵 Git 是 Linux 用户的首选😃Ubuntu 自带 Git 所以说不用自行安装了，如果想要升级的话见下：

```bash
git --version # 查看git版本

sudo add-apt-repository ppa:git-core/ppa # 添加官方源

sudo apt update && sudo apt upgrade # 如果能的话则执行更新
```

### GitHub-SSH

当然你也可以直接使用 HTTPS，但是缺点就是每次都要输入密码。而且随着 GitHub 的安全措施升级，密码也不见得是你的账号密码，而是专门的 token🥲

如此麻烦的操作是 Linux 上无法忍受的，我宁愿进行繁琐的配置，但是我一定不要每次都输入那一长串 token

这里主要参考：[设置 Git 默认推送不需要输入账号和密码【Ubuntu、SSH】](https://blog.csdn.net/qq_22841387/article/details/145183746)

```bash
git config --global user.name 'xx' # 配置全局用户名

git config --global user.email 'xxx@qq.com' # 配置全局邮箱账号

# 生成shh密钥对，然后我选择一路回车到底
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# 启动 SSH 代理并将私钥加载到代理
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa

# 查看并复制公钥内容
cat ~/.ssh/id_rsa.pub

# 在GitHub上添加这个新的ssh密钥就行

# 将现有https链接的仓库改为ssh链接
git remote rm origin
git remote add origin git@github.com:username/repository.git
```

## 安装管理软件

在 Ubuntu 上安装软件的方式大致分为以下几种;

1. 通过自带的 snap 安装
2. 通过 apt 安装
3. 通过 deb 压缩包安装
4. 通过 curl 安装

不同的安装方式有不同的管理方案，其中通过 curl 安装的管理最为不便，其他的都可以通过相应的包管理工具轻松管理

### snap

直接打开 snap 商店就可以直接看到，轻松便捷，但是其中的软件包往往较为落后

### apt

```bash
# 软件源的相关操作在桌面版中可以在"软件与更新"中进行图形化操作
# 添加软件源
sudo add-apt-repository ppa:libreoffice/ppa && sudo apt update

# 移除软件源
sudo add-apt-repository --remove ppa:libreoffice/ppa

# 安装软件
sudo apt install xxx

# 升级软件包
sudo apt update # 同步远程仓库的软件包信息，但不会实际升级任何软件
apt list --upgradable # 查看可升级的软件包
# 升级所有可用的包，但不会处理依赖关系变更（如删除旧包或安装新依赖）
sudo apt upgrade
sudo apt full-upgrade # 完全升级
sudo do-release-upgrade # 跨ubuntu大版本升级

# 查看软件包
sudo apt-cache search wps # 搜索所有包含wps的软件包及其描述信息
sudo apt-cache pkgnames | grep -i wps # 查看包含关键字wps的软件包名字

# 移除软件包
sudo apt remove xxx
sudo apt autoremove # 清理残留
```

如果出现网络访问异常，可能是 apt 的代理设置需要手动指定。apt 默认不使用系统的代理设置，需要在 `/etc/apt/apt.conf.d/99proxy` 文件中明确写入代理设置。

```bash
# 查看系统的代理设置
env | grep -i proxy

# 进入配置文件
sudo nano /etc/apt/apt.conf.d/99proxy

# 写入代理设置，请根据系统代理设置填写
Acquire::http::Proxy "http://127.0.0.1:7890/";
Acquire::https::Proxy "http://127.0.0.1:7890/";
```

### deb

从浏览器上下载 deb 压缩包之后，直接双击即可直接安装。其内部执行的命令其实就是 apt 安装，因此管理方式也是与 apt 相同的。

```bash
# 通过双击安装

# 通过apt卸载
sudo apt remove xxx
sudo apt autoremove # 清理残留
```

### AppImage

**AppImage** 是一种用于 Linux 系统的便携式软件打包格式，旨在简化应用程序的分发和运行。它的核心思想是“一个应用 = 一个文件”，用户无需安装或管理员权限即可直接运行应用程序。

在较新的 Ubuntu 版本中，直接运行该文件会出现报错，需要额外安装 `libfuse2`，使用如下命令即可：

```bash
sudo apt install libfuse2
```

然后对软件包提权：

```bash
chmod +x file_name.AppImage
```

然后双击软件包就能启动了😃


{{< alert icon="fire" cardColor="#e63946" iconColor="#ffffff" textColor="#ffffff" >}}
千万不要直接安装 fuse ，这会自动卸载 fuse3，导致新版本的 Ubuntu 文件系统崩溃！如果不小心安装了，请移除 fuse，然后查看 apt 的操作日志，将自动卸载的包手动重新安装回来！
{{< /alert >}}

如果你想卸载软件的话，也非常方便：直接把软件包删除就行了。当然，如果你和我一样有"洁癖"，可以去检查下面的这些目录，彻底清除残留：

```bash
ls ~/.config -a # 查看配置文件

ls ~/.local/share -a # 查看共享配置文件

ls ~/.cache -a # 查看缓存
du -sh ~/.cache/* | sort -h -r # 查看.cache目录下各个文件夹的磁盘占用
```

### curl

通过 curl 命令直接从目标网址下载安装脚本，然后执行这个脚本。通过 curl 安装的软件可管理性较差，原因在于：实际的安装过程是通过脚本执行的，这个过程难以监控

```bash
# 以zed编辑器的安装为例
curl -f https://zed.dev/install.sh | sh

# 如果想要卸载，一般都是难以卸载干净的
# 首先获取安装脚本文件
curl -f https://zed.dev/install.sh -o install.sh

# 把这个脚本文件丢给AI解析一下
# 然后按照AI的指令手动进行卸载
```

## 大版本更新

进行大版本的更新对于服务器 OS 来说是完全没有必要的，因为相关套件一般都还没有跟上，追逐"最新版本"并不明智。但是对于桌面版用户来说还是很有用的，毕竟更新之后就能体验最新的系统特性，说白了就是好玩儿🤓

这部分主要参考微信公众号：[如何从 Ubuntu 24.04 升级到 Ubuntu 25.04](https://mp.weixin.qq.com/s/HS95K9hohyTHNt6y9Jroew)

### 数据备份

这一步是非常有必要的，虽然说可能会占用好几十个 G 的硬盘空间，但毕竟大版本更新还是一个有风险的操作，不怕一万就怕万一😅升级成功后再删除备份，释放空间也行啊

```bash
# 安装备份工具
sudo apt install deja-dup

# 直接运行
deja-dup
```

### 更新软件包

确保系统处于最新状态可以最大程度上减少兼容性问题，逐条运行下面命令即可：

```bash
sudo apt update
sudo apt full-upgrade
sudo apt autoremove
sudo apt autoclean

sudo reboot # 重启系统
```

### 版本更新

逻辑很直接：把旧版本相关软件源指向新版本对应的软件源就行，下面罗列一些相关文件，如有修改就需要改这些文件：

- 升级策略文件：`/etc/update-manager/release-upgrades`
- 软件源配置文件：`/etc/apt/sources.list.d/ubuntu.sources`

如果你想从 LTS 版本升级为非 LTS 版本，那么就需要更改策略文件。策略文件中其实也只有一行，改成下面的样子就行：

```txt
Prompt=normal
```

接下来修改软件源配置文件，运行如下命令：

```bash
sudo sed -i 's/noble/oracular/g' /etc/apt/sources.list.d/ubuntu.sources
```

文件修改完成之后：

```bash
# 刷新索引并执行完整更新，包括内核、驱动和所有软件包
sudo apt update && sudo apt full-upgrade -y
```


{{< alert icon="pencil" cardColor="#1E3A8A" textColor="#E0E7FF" >}}
其中的 `-y` 选项是"自动确认"的意思，如果你想手动输入 yes 的话可以不加😃我反正不想
{{< /alert >}}

升级完成后：

```bash
sudo reboot # 重启系统应用更改

lsb_release -a # 验证系统版本
```

## Office套件

众所周知，Microsoft Office 是没法直接在 Linux 上直接运行的😅但是查看和编辑 `doc` 文档又是无法避免的。

因此这里推荐一个 Linux 上的 Office 平替：LibreOffice，安装方式如下：

```bash
sudo add-apt-repository ppa:libreoffice/ppa
sudo apt update
sudo apt install libreoffice
```

在安装 LibreOffice 之前也尝试过使用 WPS 来编辑 Office 文件，但是不知为何总是会引起系统报错，索性就直接弃用了


{{< alert icon="pencil" cardColor="#1E3A8A" textColor="#E0E7FF" >}}
如果你是 Office 的深度用户，换了软件就浑身难受，那么你可以尝试一下 [Wine](https://www.winehq.org/)，一个能在 Linux 上跑 Winodws 程序的神奇工具
{{< /alert >}}

## 存储清理

### 常见清理项

```bash
# 清理孤立依赖包
sudo apt autoremove

# 清理apt缓存
sudo du -sh /var/cache/apt # 查看apt缓存大小
sudo apt autoclean # 自动清理
sudo apt clean # 完全清理

# 清理系统日志
journalctl --disk-usage # 查看系统日志代大小
sudo journalctl --vacuum-time=3d # 清除三天前的日志

# 清理.cache
du -sh ~/.cache/* | sort -h -r # 查看缓存文件大小
rm -r folder_name # 直接删除即可

# 清理snap旧版本
snap list --all # 查看所有snap包

# 罗列出所有被禁用的包（下面是一行命令）
echo -e "\033[1m已禁用的 Snap 包及其占用空间:\033[0m" && snap list --all | awk '/disabled|已禁用/{print $1}' | while read -r pkg; do size=$(snap info "$pkg" | awk '/installed:/ {print $4}'); printf "%-30s %10s\n" "$pkg" "$size"; done | sort -k2 -h

# 移除所有被禁用的snap包（下面是一行命令）
snap list --all | awk '/disabled|已禁用/{print $1, $3}' | while read snapname revision; do sudo snap remove "$snapname" --revision="$revision"; done

# 清理内核
sudo dpkg --list | grep linux-image # 列出所有内核
sudo apt autoremove --purge # 自动清除不需要的内核
```

### 开机自动清理

每次都要手动检查上面这些可清理项实在是麻烦，一个聪明的电脑需要学会自己清理自己😋

首先运行命令：

```bash
sudo nano /usr/local/bin/system-clean-up.sh
```

然后把下面的文件内容粘贴进去：

```bash
#!/bin/bash
set -e

echo "[1] Running apt autoremove..."
apt autoremove -y

echo "[2] Running apt autoclean..."
apt autoclean -y

echo "[3] Cleaning journal logs older than 2 days..."
journalctl --vacuum-time=2d

echo "[4] Removing disabled snap revisions..."
snap list --all | awk '/disabled|已禁用/ {print $1, $3}' | while read snapname revision; do
  echo "Removing snap: $snapname revision $revision"
  snap remove "$snapname" --revision="$revision"
done

echo "[5] Cleaning ~/.cache/ directories larger than 200MB..."
for userdir in /home/*; do
  cache_root="$userdir/.cache"
  [ -d "$cache_root" ] || continue
  for dir in "$cache_root"/*; do
    if [ -d "$dir" ]; then
      size_kb=$(du -s "$dir" | awk '{print $1}')
      if [ "$size_kb" -gt 204800 ]; then
        echo "Removing large cache directory: $dir ($(($size_kb / 1024)) MB)"
        rm -rf "$dir"
      fi
    fi
  done
done

echo "[Done] Clean-up finished."
```

这个脚本中包含了五个可以安全地自动进行的清理项：

1. `apt autoremove`
2. `apt autoclean`
3. 清理超过两天的系统日志
4. 清理已经被禁用的 snap 包
5. 清理 `.cache` 中大于 200 MB 的缓存文件

写完了 `.sh` 脚本文件之后还需要赋予其可执行权限：

```bash
sudo chmod +x /usr/local/bin/system-clean-up.sh
```

---

然后是配置开机自启动：在目录 `/etc/systemd/system` 中存放着开机自动运行的服务脚本，后缀都是 `.service`。

为了实现开机自动清理，我们可在该目录下创建一个 `clean-up.service` 文件：

```bash
sudo nano /etc/systemd/system/clean-up.service
```

写入内容如下所示：

```bash
[Unit]
Description=Clean up system caches, logs, and snaps at boot
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/system-clean-up.sh
User=root

[Install]
WantedBy=multi-user.target
```

编辑完成后，运行如下命令来启用这个服务脚本，这样下次开机就会自动运行一次，但不会立即运行：

```bash
sudo systemctl daemon-reexec && sudo systemctl enable clean-up.service
```

配置完成后可以立即启动一次，检验脚本是否正确执行：

```bash
# 立即运行一次
sudo systemctl start clean-up.service

# 查看系统日志
sudo journalctl -u clean-up.service
```

### 自动清理配置脚本

如果你觉得上述脚本配置过程过于麻烦，这里也提供一个一键自动化配置的脚本，运行一次之后即可删除。

在任意一个目录下创建一个 `setup-cleanup.sh` 文件，写入下面的内容👇

```bash
#!/bin/bash

set -e

echo "🚀 正在创建清理脚本 /usr/local/bin/system-clean-up.sh ..."
cat << 'EOF' | sudo tee /usr/local/bin/system-clean-up.sh > /dev/null
#!/bin/bash
set -e

echo "[1] Running apt autoremove..."
apt autoremove -y

echo "[2] Running apt autoclean..."
apt autoclean -y

echo "[3] Cleaning journal logs older than 2 days..."
journalctl --vacuum-time=2d

echo "[4] Removing disabled snap revisions..."
snap list --all | awk '/disabled|已禁用/ {print $1, $3}' | while read snapname revision; do
  echo "Removing snap: $snapname revision $revision"
  snap remove "$snapname" --revision="$revision"
done

echo "[5] Cleaning ~/.cache/ directories larger than 200MB..."
for userdir in /home/*; do
  cache_root="$userdir/.cache"
  [ -d "$cache_root" ] || continue
  for dir in "$cache_root"/*; do
    if [ -d "$dir" ]; then
      size_kb=$(du -s "$dir" | awk '{print $1}')
      if [ "$size_kb" -gt 204800 ]; then
        echo "Removing large cache directory: $dir ($(($size_kb / 1024)) MB)"
        rm -rf "$dir"
      fi
    fi
  done
done

echo "[Done] Clean-up finished."
EOF

sudo chmod +x /usr/local/bin/system-clean-up.sh

echo "✅ 清理脚本创建完成"

echo "🚀 正在创建 systemd 服务 clean-up.service ..."
cat << EOF | sudo tee /etc/systemd/system/clean-up.service > /dev/null
[Unit]
Description=Clean up system caches, logs, and snaps at boot
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/system-clean-up.sh
User=root

[Install]
WantedBy=multi-user.target
EOF

echo "✅ 服务文件创建完成"

echo "🔄 重新加载 systemd..."
sudo systemctl daemon-reexec
echo "✅ systemd 重新加载完成"

echo "🧩 启用开机启动 clean-up.service ..."
sudo systemctl enable clean-up.service
echo "✅ 已启用"

echo "⚙️ 现在运行一次清理任务 ..."
sudo systemctl start clean-up.service

echo "✅ 清理完成,你可以使用 sudo journalctl -u clean-up.service 查看日志。"
```

然后在这个目录下运行安装命令：

```bash
sudo chmod +x setup-cleanup.sh && sudo ./setup-cleanup.sh
```

## 安装Nodejs

Node.js 是一个常用的环境依赖，并且常常会占用大量的硬盘空间，如果对其安装逻辑不清楚，很容易残留大量的无用文件。

我对于这种基础环境依赖的态度一直都是：如不了解，绝不安装。

从官方网站可以查询到标准的安装命令：

```bash
# 下载并安装nvm:
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash

# 当前终端重新加载nvm
\. "$HOME/.nvm/nvm.sh"

# 下载并安装 Node.js:
nvm install 22

# 检查 Node.js 版本:
node -v # 应该是 "v22.16.0".
nvm current # 应该是 "v22.16.0".

# 检查 npm 版本:
npm -v # 应该是 "10.9.2".

# 可选，如果需要安装pnpm才执行👇
# 下载并安装 pnpm:
corepack enable pnpm

# 检查 pnpm 版本:
pnpm -v
```

首先第一条命令就是使用 curl 下载 nvm 管理器，这个工具用于管理 Node.js 的版本。这个工具会被下载到 `~/.nvm` 目录下。具体的使用方法后文会有介绍。

第二条命令用于在当前命令行中加载 nvm，让 nvm 命令被有效识别。重载操作也可以不用命令进行，把命令行窗口关掉重开也行。

第三条命令就是使用 nvm 安装 Node.js。node 主要的程序包会被放置在 `~/.nvm/versions/node/` 目录中，例如 `22.16.0` 版本会被放在 `~/.nvm/versions/node/v22.16.0/` 目录中。

使用 nvm 安装 node 的时候会默认安装对应版本的 npm，因此不用特地去安装 npm🤗

pnpm 需要加一条指令才能够安装，如上所示。一些基本的 pnpm 的简介见后文：[pnpm简介]({{< relref "#pnpm简介" >}})

### nvm简介

和其他包管理工具类似，nvm 本身的使用并不复杂，其目的就是稳定忠实地完成最基本的管理工作。下面是最最常见的命令表：

功能 | 命令示例
----- | -----
安装 Node | `nvm install 22`
使用某个版本 | `nvm use 18`
设置默认版本 | `nvm alias default 18`
查看已装版本 | `nvm ls`
查看可用版本 | `nvm ls-remote`
卸载某个版本 | `nvm uninstall 16.20.2`

如果还有其他特殊的需求，可以去 nvm 的官方 GitHub 中查看：[nvm](https://github.com/nvm-sh/nvm)

### npm简介

npm 是 Node.js 的一个包管理工具，用于管理项目中的 node 依赖包。

在实际项目开发过程中，我们会使用 npm 来安装某些依赖，这些依赖包会占用大量的硬盘空间。下面是主要的清理项：

```bash
##### 第一项，项目根目录中的node_modules #####
# 在对应根目录运行👇查看空间占用
du -sh node_modules

# 清理方法简单粗暴(按需清理)
rm -r node_modules

##### 第二项，全局依赖 #####
# 查询全局依赖位置
npm root -g

# 查看全局占用大小
du -sh $(npm root -g)

# 清理方法简单粗暴(按需清理)
rm -r module_name

##### 第三项，下载缓存 #####
# 查看缓存位置
npm config get cache

# 查看缓存大小
du -sh $(npm config get cache)

# 清理缓存
npm cache clean --force
```

### pnpm简介

`pnpm` 也是一个 Node.js 包管理工具，与 `npm` 和 `yarn` 类似，但它在性能、磁盘空间占用和依赖管理方面更高效。

特性 | 说明
----- | -----
✅ 节省磁盘空间 | 使用 **内容寻址（content-addressable）** 方式，把依赖存放在共享存储中，避免重复安装
⚡ 安装速度快 | 依赖项软链接，不复制，减少 IO 操作
✅ 保证依赖隔离 | 使用严格的 `node_modules` 结构，防止隐式依赖
✅ 与 npm/yarn 兼容 | 支持大多数 npm/yarn 脚本和命令

常见命令表格，包含了与 npm 命令的对比：

操作 | `npm` 命令 | `pnpm` 替代
----- | ----- | -----
初始化项目 | `npm init` | `pnpm init`
安装依赖 | `npm install` | `pnpm install`
添加依赖 | `npm install axios` | `pnpm add axios`
移除依赖 | `npm uninstall foo` | `pnpm remove foo`
全局安装 | `npm install -g` | `pnpm add -g`
清除缓存 | `npm cache clean` | `pnpm store prune`

相比 npm，pnpm 对于空间的管理更干净，下面是一些关键目录：

位置 | 作用
----- | -----
`node_modules/` | 使用硬链接构造的虚拟依赖树
`~/.pnpm-store/` | 所有依赖包的真实文件存储位置

由于 pnpm 在 `node_modules` 中并不存放实际文件，因此也就无所谓全局与局部缓存一说。只需要一行命令就能够清理所有能够清理的依赖：

```bash
# 清理缓存
pnpm store prune
```

pnpm 会扫描所有用过的项目中的 `.pnpm-lock.yaml`，找出未再使用的版本并清理对应缓存。需要的话这行命令可以直接加入[开机自动清理]({{< relref "#开机自动清理" >}})中

## 其他

这里包含了一些简单而常用的命令

### 系统控制

- 立刻关机：`shutdown now` 或者 `systemctl poweroff`

- 立即重启：`sudo reboot` 或者 `systemctl reboot`

一般来说，使用 `systemctl` 的命令都更加"优雅"，能够让正在执行的进程自主结束后再关机，而不是直接强行杀死。

这里推荐两个使用 alias 命令来关机和重启：

```bash
alias seeu='systemctl poweroff'
alias again='systemctl reboot'
```

### 解压

命令根据你需要解压的文件格式而变动

```bash
# 解压zip文件
unzip file.zip -d /target/directory

# 解压.tar文件
tar -xvf file.tar

# 解压.tar.gz文件
tar -xzvf file.tar.gz
```

### 临时环境变量

有时即便开启了 VPN 代理，终端仍然无法正常访问外网，这时很有可能就是终端无法自动识别代理端口号导致的。解决方法就是更改环境变量即可，可以永久修改(写入文件)，也可以临时修改，如下：

```shell
# Windows中的powershell终端中使用
$env:HTTPS_PROXY="http://127.0.0.1:7890"

# bash或cmd终端中使用
set HTTPS_PROXY=http://127.0.0.1:7890
```

### 临时查看文件

一般的文件直接用对应的编辑器打开就行了。但是有一些特殊文件，例如超大的 csv 文件，只想快速查看一小部分数据。这个时候就可以使用 `head` 命令：

```bash
# 读取your_file文件的前10行并显示(不会读取完整文件)
head -n 10 your_file.csv
```

如果出现乱码，一定是编码格式出错了，可以使用 `iconv` 进行转换：

```bash
# 从gbk转为utf-8
head -n 10 "your_file.csv" | iconv -f gbk -t utf-8
```

如果你不清楚到底是什么编码转什么编码，可以直接把乱码丢给 AI，简单方便😋

## 引用文献

- [如何在Ubuntu系统中进行磁盘的分区与挂载](https://cloud.tencent.com/developer/article/2456171)
- [LibreOffice Suite - 适用于 Linux 的 Microsoft Office 套件的最佳替代方案](https://cn.linux-terminal.com/?p=1602)
- [在 Ubuntu 安装配置 Fcitx 5 中文输入法](https://muzing.top/posts/3fc249cf/)
- [设置 Git 默认推送不需要输入账号和密码【Ubuntu、SSH】](https://blog.csdn.net/qq_22841387/article/details/145183746)
