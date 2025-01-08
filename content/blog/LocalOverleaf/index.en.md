---
title: Local OverLeaf Deployment
weight: -5
draft: false
description: Complete process of local Overleaf deployment
slug: localoverleaf
tags:
  - overleaf
  - LaTeX
series:
  - Operation
series_order: 1
date: 2024-07-12
authors:
  - Morethan
---

## Background Overview

- You should know how to interact with the computer via the command line, including but not limited to: how to open the command line/terminal in Windows, when a running command ends, etc.

- A little knowledge of bypassing internet censorship is helpful. OverLeaf is foreign software, and its related LaTeX projects are also hosted abroad. Therefore, directly accessing foreign traffic when downloading dependencies can save a lot of trouble. If you don’t have a VPN, you will need to specify a domestic source for each package manager, though sometimes the updates from domestic sources are not timely.

- Basic familiarity with Vim operations is useful, such as: how to enter insert mode, how to save and exit, how to exit without saving, etc.

## Full Deployment Process

### Installing Linux
Search for a Linux distribution in the `Windows App Store` and download it. The author chose `Kali`. After installation, you can open it directly from the Start menu. Upon opening, a command-line window will pop up, and you will need to register with a username and password.

{{< alert >}}
At this point, your command line should display a warning. This is because you haven’t installed WSL (Windows Subsystem for Linux).

Also, when entering the password, your input will not be displayed in the command line, but it has been recorded.
{{< /alert >}}

Why do you need a Linux system? Because OverLeaf's ShareLaTeX model requires a Linux environment. It is said that OverLeaf runs more smoothly on Linux systems.

### Installing WSL
To install WSL2, run the following in the Windows command line:
```sh
wsl --install
```

After installation, you can open it directly. Another warning will appear. At this point, you need to create a text file in the C:\Users\ASUS directory and rename it to `.wslconfig`.

Enter the following content:
```txt
[experimental] autoMemoryReclaim=gradual # gradual | dropcache | disabled networkingMode=mirrored dnsTunneling=true firewall=true autoProxy=true
```

### Installing Docker
Go to the Docker website to download Docker, which will be the container for the ShareLaTeX model. Docker is an open-source application container engine that includes images, containers, and repositories. Its purpose is to manage the lifecycle of application components, such as encapsulation, distribution, deployment, and operation, allowing users to "package once, run anywhere," much like a container, developed and encapsulated by programmers, which users can directly move around.

Once Docker is installed, you can double-click to start it in the background. We will interact with Docker later via the command line.

### Pulling the Image

Open `Kali`, and run the following command:
```sh
git clone https://github.com/overleaf/toolkit.git ./overleaf-toolkit
```

Then run:
```sh
cd ./overleaf-toolkit
bin/init
vim ./config/variables.env
```

At this point, you should be in the document interface of the Vim text editor. Vim has many shortcuts, and pressing the `"I"` key will enter insert mode for text editing. Press `"esc"` to return to normal mode. In insert mode, type: `OVERLEAF_SITE_LANGUAGE=zh-CN`.

After typing, press `"esc"` to return to normal mode, then type `:wq` to "save and quit." If you make a mistake, type `:e!` to discard all changes and start over. This step will set your OverLeaf interface to Chinese.

After successfully saving and quitting, return to the familiar `Kali` command-line interface and run `bin/up`. This will pull the ShareLaTeX image and related network tools. There will be a large amount of data transfer, so ensure that your network is stable (your VPN should be reliable!).

### Configuring the User
Once the previous command finishes, run `bin/start`. At this point, open Docker and enter the ShareLaTeX container. You should see code "flashing." If there are no red messages, everything is running normally.

Now open a browser and visit `http://localhost/launchpad`.

After registering an Administrator Account, you will be redirected to `http://localhost/project`. The basic OverLeaf webpage should now be displayed.
{{< alert >}}
If you compile now, it will most likely report an error `ᕕ( ᐛ )ᕗ`.

This is because ShareLaTeX is missing many required packages🙃"
{{< /alert >}}

### Installing Extension Packages

Open `Kali`, navigate to the appropriate directory, and run `bin/shell`. Then execute the following one by one:
```sh
cd /usr/local/texlive

# Download and run the upgrade script
wget http://mirror.ctan.org/systems/texlive/tlnet/update-tlmgr-latest.sh
sh update-tlmgr-latest.sh -- --upgrade

# Change the TeX Live download source
tlmgr option repository https://mirrors.sustech.edu.cn/CTAN/systems/texlive/tlnet/

# Upgrade tlmgr
tlmgr update --self --all

# Install the full TeX Live package (this will take time, so don’t let the shell disconnect)
tlmgr install scheme-full

# Exit the ShareLaTeX command-line interface
exit

# Restart the ShareLaTeX container
docker restart sharelatex
```

After restarting, enter the `shell` again and run:
```sh
apt update

# Install fonts
apt install --no-install-recommends ttf-mscorefonts-installer fonts-noto texlive-fonts-recommended tex-gyre fonts-wqy-microhei fonts-wqy-zenhei fonts-noto-cjk fonts-noto-cjk-extra fonts-noto-color-emoji fonts-noto-extra fonts-noto-ui-core fonts-noto-ui-extra fonts-noto-unhinted fonts-texgyre

# Install pygments
apt install python3-pygments

# Install Beamer and others
apt install texlive-latex-recommended
apt install texlive-latex-extra

# Install English fonts
echo "yes" | apt install -y --reinstall ttf-mscorefonts-installer

# Install Chinese fonts
apt install -y latex-cjk-all texlive-lang-chinese texlive-lang-english
cp fonts/* /usr/share/fonts/zh-cn/
cd /usr/share/fonts
fc-cache -fv # Update font cache
fc-list :lang=zh-cn
fc-match Arial
```

Finally, in the `shell` directory, run:
```sh
vim /usr/local/texlive/2023/texmf.cnf
```
Open the configuration file and add `shell_escape = t` at the bottom.


{{< alert icon="circle-info" >}}
I’m not sure what this does, but it was passed down by the predecessors 🤔
{{< /alert >}}

Note, if the TeX Live version (the official name for extension packages) differs, the directory path may also change. You will need to adjust the path based on the actual version, for example, change `2023` to `2024`.

{{< alert icon="circle-info" >}}
You can use `ls -l` in the Linux command line to view all files in the current directory.
{{< /alert >}}
## Successful Deployment
Now you can happily use your local OverLeaf version without worrying about compilation timeouts~

If you're lucky and happen to be a CQUer, here’s a graduation thesis template from Chongqing University, super user-friendly: [CQUThesis](https://github.com/nanmu42/CQUThesis)
