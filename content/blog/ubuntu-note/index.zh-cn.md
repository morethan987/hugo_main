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
lastmod: 2025-06-20
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
这个命令行编辑器是 Umbundu 系统自带的命令行文本编辑器，不用专门下载。

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

## 其他

这里包含了一些简单而常用的命令

### 系统控制

- 立刻关机：`shutdown now`

- 立即重启：`sudo reboot`

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

## 引用文献

- [如何在Ubuntu系统中进行磁盘的分区与挂载](https://cloud.tencent.com/developer/article/2456171)
- [LibreOffice Suite - 适用于 Linux 的 Microsoft Office 套件的最佳替代方案](https://cn.linux-terminal.com/?p=1602)
- [在 Ubuntu 安装配置 Fcitx 5 中文输入法](https://muzing.top/posts/3fc249cf/)
- [设置 Git 默认推送不需要输入账号和密码【Ubuntu、SSH】](https://blog.csdn.net/qq_22841387/article/details/145183746)
