---
title: Ubuntu折腾札记
weight: -85
draft: false
description: 记录折腾Ubuntu系统的过程以备不时之需
slug: ubuntu-note
tags:
  - Ubuntu
  - Linux
series:
  - 技术杂项
series_order: 7
date: 2025-05-01
lastmod: 2025-05-01
authors:
  - Morethan
---

{{< lead >}}
总结记录一下折腾Ubuntu系统的过程，以备不时之需
{{< /lead >}}

## 前言

Ubuntu 作为一个流行的 Linux 发行版本，对比其他 Linux 发行版，有较好的生态支持。最主要的体现就是：当你遇到问题的时候 Ubuntu 能够搜到的教程更多。

这篇文章主要针对带有 GUI 的个人版 Ubuntu 系统，服务器专用的 Ubuntu 系统操作取决于你的实际业务，[云服务器搭建]({{< ref "/blog/cloud-server-build/" >}})这篇文章可以作为一个参考。

## Ubuntu安装

非常久远之前就安装好了，故不做记录了。如有需要，请自行搜索“移动硬盘安装 Ubuntu”等关键词。这里放一个较新的链接：[移动硬盘安装Ubuntu](https://blog.csdn.net/qq_52034548/article/details/131581118)

安装过程已经不可考究了，现状就是我将 Ubuntu 系统安装在一个移动硬盘中，随插随用。

使用的时候就可以在开机之前把硬盘插上，点击电源按钮之后快速点击某个按键进入 BIOS 引导界面，把优先级提升到最高然后保存推出就可以正常进入 Ubuntu 系统了。

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
ln -s /target/dir ~/Desktop

# 测试，如果能cd进去那么就没问题
cd ~/Desktop/dir
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
# 安装软件
sudo apt install xxx

# 升级软件包
sudo apt update # 同步远程仓库的软件包信息，但不会实际升级任何软件
apt list --upgradable # 查看可升级的软件包
# 升级所有可用的包，但不会处理依赖关系变更（如删除旧包或安装新依赖）
sudo apt upgrade
sudo apt full-upgrade # 完全升级
sudo do-release-upgrade # 跨ubuntu大版本升级

# 移除软件包
sudo apt remove xxx
sudo apt autoremove # 清理残留
```

### deb

从浏览器上下载 deb 压缩包之后，直接双击即可直接安装。其内部执行的命令其实就是 apt 安装，因此管理方式也是与 apt 相同的。

```bash
# 通过双击安装

# 通过apt卸载
sudo apt remove xxx
sudo apt autoremove # 清理残留
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

## 存储清理

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

# 清除缩略图
sudo du -sh ~/.cache/thumbnails # 查看缩略图的占用空间
sudo rm -rf ~/.cache/thumbnails/* # 可安全清除，会自动重建

# 清理snap旧版本
snap list --all # 查看所有snap包
# 罗列出所有被禁用的包（下面是一行命令）
echo -e "\033[1m已禁用的 Snap 包及其占用空间:\033[0m" && snap list --all | awk '/disabled|已禁用/{print {{< katex >}}\\(1}' | while read -r pkg; do size=\\)(snap info "{{< katex >}}\\(pkg" | awk '/installed:/ {print\\)4}'); printf "%-30s %10s\n" "{{< katex >}}\\(pkg" "\\)size"; done | sort -k2 -h && echo -e "\n\033[1m总占用空间: {{< katex >}}\\((snap list --all | awk '/disabled|已禁用/{print\\)1}' | xargs -I{} snap info {} | awk '/installed:/ {sum += {{< katex >}}\\(3} END {print sum/1024 "MB"}')\\033[0m" # 移除所有被禁用的snap包（下面是一行命令） snap list --all | awk '/disabled|已禁用/{print\\)1, {{< katex >}}\\(3}' | while read snapname revision; do snap remove "\\)snapname" --revision="$revision"; done

# 清理内核
sudo dpkg --list | grep linux-image # 列出所有内核
sudo apt autoremove --purge # 自动清除不需要的内核
```

## 引用文献

- [如何在Ubuntu系统中进行磁盘的分区与挂载](https://cloud.tencent.com/developer/article/2456171)
- 