---
title: Ubuntu Tinkering Notes
weight: -90
draft: false
description: Documenting the process of tinkering with Ubuntu for future reference
slug: ubuntu-note
language: en
tags:
  - Ubuntu
  - Linux
series:
  - Technical Miscellany
series_order: 8
date: 2025-05-01
lastmod: 2025-06-21
authors:
  - Morethan
---

{{< lead >}} Summarizing and documenting the process of tinkering with Ubuntu for future reference.   {{< /lead >}}

## Introduction  

Ubuntu, as a popular Linux distribution, offers better ecosystem support compared to other Linux distros. The most notable advantage is that when you encounter issues, you're more likely to find tutorials and solutions for Ubuntu.  

This article primarily focuses on the GUI-based personal edition of Ubuntu. For server-specific Ubuntu systems, operations depend on your actual business needs. The article [Cloud Service Deployment]({{< ref "/blog/cloud-server-build/" >}}) can serve as a reference.  

## Ubuntu Installation  

The installation was done a long time ago, so no detailed records exist. If needed, please search for keywords like "installing Ubuntu on a portable hard drive." Here’s a relatively recent guide: [Installing Ubuntu on a Portable Hard Drive](https://blog.csdn.net/qq_52034548/article/details/131581118).  

The installation process is no longer traceable. The current setup involves installing Ubuntu on a portable hard drive, allowing it to be used on-the-go.  

To use it, simply plug in the hard drive before powering on the computer. Quickly press a specific key to enter the BIOS boot menu, set the priority to the highest, save, and exit to boot into Ubuntu.  

To switch back to Windows, just unplug the hard drive and power on normally—no additional steps are required.  

## Interface Customization  

I personally prefer an Apple-style interface, so I specifically chose an Apple-inspired theme: [WhiteSur](https://github.com/vinceliuice/WhiteSur-gtk-theme).  

The installation process is straightforward:  

```bash
git clone https://github.com/vinceliuice/WhiteSur-gtk-theme.git --depth=1  

cd WhiteSur-gtk-theme  

./install.sh # Run the installation script  
```

For detailed configuration, refer to the instructions on the GitHub page.  

As for updates, the official guide doesn’t specify, presumably assuming users already know:  

```bash
git pull # Fetch the latest code  

./install.sh # Re-run the installation script  
```

## Text Editing

Text editing here specifically refers to editing text in the command line. Editing text in a dedicated text editor is so straightforward and simple that it doesn't require learning. And in most cases, we don't need to edit text in the command line.

However, there are times when you have to edit text in the command line: servers often lack a GUI😢 Even though you can run a GUI editor on a server using VS Code plugins, launching such a "heavyweight" editor just to modify a configuration file feels like overkill🤔

If you simply want to create a file and then edit it with another editor, one simple command will suffice:

```bash
# Just touch it and it will appear😉
touch file_name.md
```

### nano

This command-line editor is pre-installed in Ubuntu systems, so there's no need for a separate download.

Editing a file is very simple:

```bash
nano file_name
```

Once inside nano, there are a few lines of prompts that highlight the relevant shortcuts; just follow the instructions. It’s a very simple command-line text editor.

### vim

I've heard about this "legendary ancient" editor for a long time and learned about its many advantages: keeping your fingers on the keyboard helps with thought flow, once you get used to it you can't quit, it's minimalist and highly customizable, all cool programmers use vim…

After several attempts, I came to the conclusion: vim is great, but I'm really lazy😢 I really don't want to spend weeks getting used to all kinds of key combinations.

Here I only document the most commonly used operations, suitable for those who occasionally use vim for lightweight editing. After mastering these operations, you can claim that you know how to use vim😋

```bash
i # Enter insert mode

<ESC> # Return to normal mode

: # Enter command mode

:q! # Quit without saving

:wq # Save and quit

y6w # Copy 6 words

p # Paste

dd # Delete a line

dw # Delete a word
```


{{< alert icon="pencil" cardColor="#1E3A8A" textColor="#E0E7FF" >}}
There are many detailed vim tutorials online. If you want to learn vim thoroughly, you can check them out👀 It's clear that those tutorials really want me to become a vim master😔 But I'm just too lazy😔
{{< /alert >}}

### neovim

Due to various unavoidable reasons, I occasionally have to use a terminal IDE for project development 😢. However, the traditional Vim editor is just too "hardcore". I recently heard that Neovim is quite user-friendly and decided to give it a try.  

**Neovim** is a modern reimplementation of Vim, short for *Neo (New) Vim*. Its goal is to bring modern features to the classic Vim editor, including:

- A better plugin system (asynchronous processing, Lua support)
- Enhanced language server integration (LSP)
- More user-friendly configuration options (using Lua instead of VimScript)
- Better integration with IDE-like features (debugging, completion, navigation, etc.)

#### Directory Structure

Directory | Location | Purpose
----- | ----- | -----
`~/.config/nvim/` | Main configuration directory | Stores `init.lua/init.vim` and custom plugin configurations
`~/.local/share/nvim/` | Data directory | Plugin repositories, automatically downloaded plugins (created by plugin managers)
`~/.cache/nvim/` | Cache directory | Stores caches, LSP logs, Telescope indexes, Tree-sitter caches, etc.
`~/.local/state/nvim/` | State directory | Runtime temporary states (crash records, in-process data)
`~/.config/nvim/lazy/` | Plugin configuration | Plugin loading information generated by plugin managers

Main configuration directory structure:

```text
~/.config/nvim/
├── init.lua                 # Entry file, similar to VS Code's settings.json
├── lua/                     # Contains all your Lua configuration modules
│   ├── core/                # General settings, keymaps, custom functions, etc.
│   ├── plugins/             # Plugin configuration files (one file per plugin)
│   └── lazy.lua             # Initialization file for lazy.nvim plugin loader
├── after/                   # Additional settings executed after plugins are loaded
│   └── plugin/              # E.g., filetype autocommands
└── plugin/                  # Lua/Vim scripts that are auto-loaded
```

#### Configuration Solutions

Nowadays, many incredibly sleek terminal IDEs are actually powered by Neovim under the hood. These IDEs are essentially pre-configured setups crafted by Neovim enthusiasts—essentially the results of their meticulous tinkering. Some well-known examples include: [LazyVim](https://github.com/LazyVim/LazyVim) and [NvChad](https://github.com/NvChad/NvChad).  

These third-party configurations usually come with comprehensive documentation, so you can simply follow the official instructions to install them.

## Fcitx 5  

Main reference: [Install and Configure Fcitx 5 Chinese Input Method on Ubuntu](https://muzing.top/posts/3fc249cf/). I initially considered using Sogou Input Method, but the official installation process seemed overly complicated, and it also required installing Fcitx 5. So, I figured I might as well just use Fcitx 5 directly.  

To be honest, I was reluctant to use Fcitx at first 🥲 because its interface is so "plain" that it’s hard to accept. I believe the author of the blog post above must have felt the same way 👆.

## Using Windows Fonts  

Approach: Copy font files from Windows to Ubuntu’s dedicated font directory, assign appropriate permissions, refresh Ubuntu’s font cache, and load the new fonts.  

```bash
# Windows font directory: C:/Windows/Fonts  
sudo cp /mnt/C/Windows/Fonts/LXGWWenKai-Regular.ttf /usr/share/fonts/custom/LXGWWenKai-Regular.ttf  

# Grant permissions  
sudo chmod u+rwx /usr/share/fonts/custom/*  

# Navigate to the font directory  
cd /usr/share/fonts/custom/  

# Create font cache  
sudo mkfontscale  

# Refresh cache  
sudo fc-cache -fv  
```

Alternatively, you can download a new `.ttf` file from the web and copy it to the target directory. If you’re using a GUI-based Ubuntu system, you can simply double-click the font file to install it 🥰.  


{{< alert icon="triangle-exclamation" cardColor="#ffcc00" textColor="#333333" iconColor="#8B6914" >}}
Fonts may be installed redundantly, as the system doesn’t check for duplicates. If this happens, manually locate and delete the duplicate files in the relevant directory 🥲. Ubuntu’s font tools can display all font information.  
{{< /alert >}}

## Setting Up VPN

I usually use Clash + airport to access external network resources.

First, of course, follow the airport tutorial to download Clash and obtain the configuration file. Then, a VPN is typically started using a command like this:

```bash
./clash -d .
```

However, this command occupies a terminal and requires manual input every time after booting, which is quite inconvenient.

Therefore, this command can be encapsulated into a system service, allowing it to be controlled and managed using `systemctl`. It can also be set to start automatically on boot.

Sample configuration file, which needs to be written to `/etc/systemd/system/clash.service`:

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

Set it to start automatically on boot:

```bash
sudo systemctl daemon-reexec && sudo systemctl enable clash.service
```

Control it using `systemctl`:

```bash
# Start immediately
sudo systemctl start clash.service

# Stop immediately
sudo systemctl stop clash.service
```

Of course, you can also manually configure it in: `System Settings --> Network --> Proxy`.

## Mounting Hard Drives  

Since my Ubuntu system is installed on a portable hard drive, the main goal here is to access Windows partitions from Ubuntu. This section doesn’t cover detailed partition operations. For tasks like formatting partitions, refer to: [How to Partition and Mount Disks in Ubuntu](https://cloud.tencent.com/developer/article/2456171).  

```bash
# View disks and partitions (sudo privileges required)  
sudo fdisk -l  

# Create a mount point (essentially creating a folder)  
# The subfolder under /mnt is named "E" because it’s intended to mount the E drive  
sudo mkdir /mnt/E  

# Mount the new partition directly  
sudo mount /dev/vdb /mnt/E  

# Set auto-mount at boot  
# Check the partition’s UUID  
sudo blkid  

# Edit the specific file  
vim /etc/fstab  

# Append to the end of the file  
UUID=xxxxxxxx /mnt/E ntfs defaults 0 2  
```

- Replace the UUID above with the output from `blkid`. Replace `ntfs` with the appropriate filesystem type (common types include `ntfs` and `ext 4`).  
- `defaults`: This is a combination of default mount options, such as `rw` (read-write) and `relatime` (reduces inode access time updates).  
- `0` and `2`: These values control backup and filesystem check order. Typically, the first value is `0` (no backup), and the second is `1` or `2` (`1` for the root filesystem, `2` for others).  

**Testing method**:  

```bash
# If no errors occur, the configuration is correct  
sudo mount -a  
```

## Creating Shortcuts  

A common task: placing a quick link to a frequently used folder on the desktop for easy access.  

```bash
# Place a link to /target/dir in the Desktop folder  
# Replace with your target directory  
ln -s /target/dir ~/Desktop  

# Test—if you can cd into it, it works  
cd ~/Desktop/dir  
```

## System Information

When there is no UI, you need to use the command line to check relevant system information.

```bash
# Summary of system information
hostnamectl

# CPU details
lscpu

# Check CPU usage
top

# Partition usage
df -h
```

## Configuring Git

One of the standout features of Linux is its extreme simplicity, which is why using the command line to manage Git is the preferred choice for Linux users 😃. Ubuntu comes with Git pre-installed, so there's no need to install it separately. If you want to upgrade, follow these steps:

```bash
git --version # Check the Git version

sudo add-apt-repository ppa:git-core/ppa # Add the official repository

sudo apt update && sudo apt upgrade # If possible, proceed with the upgrade
```

### GitHub-SSH

Of course, you can also use HTTPS directly, but the downside is that you’ll need to enter your password every time. Moreover, with GitHub's increasing security measures, the password isn’t necessarily your account password but rather a dedicated token 🥲.

Such a cumbersome process is unbearable on Linux. I’d rather go through a tedious setup once than have to enter a long token every time.

This section is mainly referenced from: [Configuring Git to Push by Default Without Entering Credentials (Ubuntu, SSH)](https://blog.csdn.net/qq_22841387/article/details/145183746).

```bash
git config --global user.name 'xx' # Configure the global username

git config --global user.email 'xxx@qq.com' # Configure the global email account

# Generate an SSH key pair. Here, I choose to press Enter all the way through.
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# Start the SSH agent and load the private key into the agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa

# View and copy the public key content
cat ~/.ssh/id_rsa.pub

# Add this new SSH key to your GitHub account.

# Change an existing HTTPS-linked repository to an SSH link
git remote rm origin
git remote add origin git@github.com:username/repository.git
```

## Installing and Managing Software  

Software installation on Ubuntu generally falls into the following categories:  

1. Via the built-in Snap store  
2. Via `apt`  
3. Via `.deb` packages  
4. Via `curl`  

Different installation methods require different management approaches. `curl` installations are the most cumbersome to manage, while others can be handled easily with their respective package managers.  

### Snap  

Simply open the Snap store to install software effortlessly, though the packages are often outdated.  

### apt  

```bash
# Software source operations can be performed graphically in the desktop version via "Software & Updates"
# Add a software source
sudo add-apt-repository ppa:libreoffice/ppa && sudo apt update

# Remove a software source
sudo add-apt-repository --remove ppa:libreoffice/ppa

# Install software  
sudo apt install xxx  

# Update packages  
sudo apt update # Sync package info from remote repositories without upgrading  
apt list --upgradable # View upgradable packages  
# Upgrade all available packages without handling dependency changes  
sudo apt upgrade  
sudo apt full-upgrade # Full upgrade  
sudo do-release-upgrade # Upgrade across major Ubuntu versions  

# Check software packages
sudo apt-cache search wps # Search for all packages containing "wps" and their description information
sudo apt-cache pkgnames | grep -i wps # View package names containing the keyword "wps"

# Remove packages  
sudo apt remove xxx  
sudo apt autoremove # Clean up residuals  
```

If there is an abnormality in network access, it might be necessary to manually specify the proxy settings for apt. By default, apt does not use the system's proxy settings; instead, you need to explicitly write the proxy settings into the `/etc/apt/apt.conf.d/99proxy` file.

```bash
# Check the system's proxy settings
env | grep -i proxy

# Open the configuration file
sudo nano /etc/apt/apt.conf.d/99proxy

# Write the proxy settings, filling them out according to your system's proxy settings
Acquire::http::Proxy "http://127.0.0.1:7890/";
Acquire::https::Proxy "http://127.0.0.1:7890/";
```

### deb  

After downloading a `.deb` package from a browser, double-clicking it will install it directly. Internally, this uses `apt`, so management is the same as with `apt`.  

```bash
# Install via double-click  

# Uninstall via apt  
sudo apt remove xxx  
sudo apt autoremove # Clean up residuals  
```

### AppImage  

**AppImage** is a portable software packaging format for Linux systems, designed to simplify application distribution and execution. Its core philosophy is "one app = one file," allowing users to run applications directly without installation or administrator privileges.

In newer versions of Ubuntu, attempting to run the file directly may result in an error. To resolve this, you need to install `libfuse2` using the following command:

```bash
sudo apt install libfuse2
```

Then grant executable permissions to the AppImage package:

```bash
chmod +x file_name.AppImage
```

After installation, you can simply double-click the package to launch the application 😃.


{{< alert icon="fire" cardColor="#e63946" iconColor="#ffffff" textColor="#ffffff" >}}
**Never install fuse directly**, as this will automatically uninstall fuse3, causing the file system in newer Ubuntu versions to crash! If you accidentally install it, remove fuse and check the apt operation log to manually reinstall any automatically removed packages.
{{< /alert >}}

If you want to uninstall the software, it’s very straightforward: just delete the package. However, if you’re a perfectionist like me, you can check the following directories to completely clean up any residual files:

```bash
ls ~/.config -a # Check configuration files

ls ~/.local/share -a # Check shared configuration files

ls ~/.cache -a # Check cache
du -sh ~/.cache/* | sort -h -r # Check disk usage of folders under .cache
```

### curl  

Download and execute installation scripts directly from a URL using `curl`. Software installed this way is harder to manage because the actual installation process is script-driven and difficult to monitor.  

```bash
# Example: Installing the Zed editor  
curl -f https://zed.dev/install.sh | sh  

# Uninstalling is usually messy  
# First, fetch the installation script  
curl -f https://zed.dev/install.sh -o install.sh  

# Have an AI parse the script  
# Then follow the AI’s instructions to manually uninstall  
```

## Major Version Updates  

Performing major version updates is completely unnecessary for server OSes, as the related software packages usually haven't caught up yet. Chasing the "latest version" isn’t wise. However, for desktop users, it’s quite useful—after all, updating allows them to experience the newest system features. In short, it’s just for fun. 🤓  

This section primarily references the WeChat public article: [How to Upgrade from Ubuntu 24.04 to Ubuntu 25.04](https://mp.weixin.qq.com/s/HS95K9hohyTHNt6y9Jroew).  

### Data Backup  

This step is essential. Although it might take up dozens of gigabytes of disk space, a major version update is still a risky operation. Better safe than sorry. 😅 You can always delete the backup and free up space after a successful upgrade.  

```bash
# Install the backup tool  
sudo apt install deja-dup  

# Run directly  
deja-dup  
```

### Update Software Packages  

Ensuring the system is up to date minimizes compatibility issues. Execute the following commands one by one:  

```bash
sudo apt update  
sudo apt full-upgrade  
sudo apt autoremove  
sudo apt autoclean  

sudo reboot # Reboot the system  
```

### Version Upgrade  

The logic is straightforward: point the old version’s software sources to those of the new version. Below are some relevant files that may need modification:  

- Upgrade policy file: `/etc/update-manager/release-upgrades`  
- Software sources configuration file: `/etc/apt/sources.list.d/ubuntu.sources`  

If you want to upgrade from an LTS version to a non-LTS version, you'll need to modify the policy file. The policy file actually contains just one line—change it to the following:  

```txt
Prompt=normal  
```

Next, modify the software source configuration file by running the following command:  

```bash
sudo sed -i 's/noble/oracular/g' /etc/apt/sources.list.d/ubuntu.sources  
```

After modifying the files:

```bash
# Refresh the index and perform a full upgrade, including the kernel, drivers, and all packages  
sudo apt update && sudo apt full-upgrade -y  
```


{{< alert icon="pencil" cardColor="#1E3A8A" textColor="#E0E7FF" >}}
The `-y` option means "automatic confirmation." If you prefer to manually type "yes," you can omit it. 😃 Personally, I’d rather not.  
{{< /alert >}}

After the upgrade completes:  

```bash
sudo reboot # Reboot to apply changes  

lsb_release -a # Verify the system version  
```

## Office Suite  

As we all know, Microsoft Office cannot run directly on Linux 😅. However, viewing and editing `doc` files is often unavoidable.  

Therefore, here’s a recommended Office alternative for Linux: **LibreOffice**. The installation steps are as follows:

```bash
sudo add-apt-repository ppa:libreoffice/ppa  
sudo apt update  
sudo apt install libreoffice  
```

Before installing LibreOffice, I also tried using **WPS** to edit Office files, but for some reason, it kept causing system errors, so I eventually abandoned it.


{{< alert icon="pencil" cardColor="#1E3A8A" textColor="#E0E7FF" >}}
If switching away from Office makes you feel lost, [Wine](https://www.winehq.org/) might be your savior—it’s the sorcery that runs Windows apps on Linux!  
{{< /alert >}}

## Storage Cleanup  

### Common Cleanup Tasks

```bash
# Remove orphaned dependencies  
sudo apt autoremove  

# Clear apt cache  
sudo du -sh /var/cache/apt # Check apt cache size  
sudo apt autoclean # Auto-clean  
sudo apt clean # Full clean  

# Clear system logs  
journalctl --disk-usage # Check system log size  
sudo journalctl --vacuum-time=3 d # Remove logs older than 3 days  

# Clear .cache
du -sh ~/.cache/* | sort -h -r # Check cache size
rm -r folder_name # delete the folder recursively

# Clean up old Snap versions
snap list --all # List all Snap packages

# List all disabled packages (single-line command)
echo -e "\033[1 mDisabled Snap Packages and Their Sizes:\033[0 m" && snap list --all | awk '/disabled|已禁用/{print $1}' | while read -r pkg; do size=$(snap info "$pkg" | awk '/installed:/ {print $4}'); printf "%-30 s %10 s\n" "$pkg" "$size"; done | sort -k 2 -h

# Remove all disabled Snap packages (single-line command)  
snap list --all | awk '/disabled|已禁用/{print $1, $3}' | while read snapname revision; do sudo snap remove "$snapname" --revision="$revision"; done

# Clean up old kernels  
sudo dpkg --list | grep linux-image # List all kernels  
sudo apt autoremove --purge # Automatically remove unnecessary kernels  
```

### Auto Clean on Boot

Manually checking and cleaning up your system every time can be a hassle. A smart computer should learn to clean itself 😋

Run the following command to open a new script file:

```bash
sudo nano /usr/local/bin/system-clean-up.sh
```

Then paste the following content into the file:

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

This script includes five safe automatic cleanup tasks:

1. `apt autoremove`
2. `apt autoclean`
3. Deletes system logs older than 2 days
4. Removes disabled Snap packages
5. Deletes `.cache` subfolders larger than 200MB

After saving the file, grant execute permission:

```bash
sudo chmod +x /usr/local/bin/system-clean-up.sh
```

---

Then configure automatic startup on boot: The directory `/etc/systemd/system` contains service scripts that run automatically at startup, all with the `.service` extension.  

To enable the script to run automatically at system boot, create a new systemd service file:

```bash
sudo nano /etc/systemd/system/clean-up.service
```

Paste in the following content:

```ini
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

After editing is completed, run the following command to enable this service script, so it will automatically run once upon the next boot but will not run immediately.

```bash
sudo systemctl daemon-reexec && sudo systemctl enable clean-up.service
```

You can also run it manually to test if it works properly:

```bash
# Run immediately
sudo systemctl start clean-up.service

# Check the system journal
sudo journalctl -u clean-up.service
```

### One-Click Auto Setup Script

If you find the above script configuration process too cumbersome, we also provide a one-click automated setup script. You can delete it after running it once.

Create a file named `setup-cleanup.sh` in any directory and paste the following content:

```bash
#!/bin/bash

set -e

echo "🚀 Creating cleanup script at /usr/local/bin/system-clean-up.sh..."
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

echo "✅ Cleanup script created."

echo "🚀 Creating systemd service clean-up.service..."
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

echo "✅ Service file created."

echo "🔄 Reloading systemd..."
sudo systemctl daemon-reexec
echo "✅ systemd reloaded."

echo "🧩 Enabling clean-up.service at boot..."
sudo systemctl enable clean-up.service
echo "✅ Enabled."

echo "⚙️ Running cleanup task now..."
sudo systemctl start clean-up.service

echo "✅ Cleanup complete. You can check logs with: sudo journalctl -u clean-up.service"
```

Finally, run this setup script once and you're done:

```bash
sudo chmod +x setup-cleanup.sh && sudo ./setup-cleanup.sh
```

## Install Nodejs

Node.js is a commonly used environment dependency and often takes up a lot of disk space. If you don't understand its installation logic, it's easy to leave behind a large number of useless files.

My attitude towards such basic environment dependencies has always been: if you don't understand it, never install it.

The standard installation command can be found on the official website:

```bash
# Download and install nvm:
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash

# Reload nvm in the current terminal
. "$HOME/.nvm/nvm.sh"

# Download and install Node.js:
nvm install 22

# Check Node.js version:
node -v # Should be "v22.16.0".
nvm current # Should be "v22.16.0".

# Check npm version:
npm -v # Should be "10.9.2".

# Optional, execute only if pnpm needs to be installed👇
# Download and install pnpm:
corepack enable pnpm

# Check pnpm version:
pnpm -v
```

The first command uses curl to download the nvm manager, which is a tool for managing Node.js versions. This tool will be downloaded to the `~/.nvm` directory. Specific usage methods will be introduced later.

The second command is used to load nvm in the current command line so that the nvm command can be effectively recognized. The reload operation can also be done without a command by simply closing and reopening the command-line window.

The third command uses nvm to install Node.js. The main node packages will be placed in the `~/.nvm/versions/node/` directory, for example, the `22.16.0` version will be placed in the `~/.nvm/versions/node/v22.16.0/` directory.

When using nvm to install node, the corresponding version of npm will be installed by default, so there is no need to install npm separately 😊

pnpm requires an additional command to install, as shown above. A brief introduction to pnpm is provided below: [Introduction to pnpm]({{< relref "#introduction-to-pnpm" >}})

### Introduction to nvm

Similar to other package management tools, the use of nvm itself is not complicated, and its purpose is to stably and faithfully complete the most basic management work. Below is the most common command table:

Function | Command Example
----- | -----
Install Node | `nvm install 22`
Use a specific version | `nvm use 18`
Set default version | `nvm alias default 18`
View installed versions | `nvm ls`
View available versions | `nvm ls-remote`
Uninstall a specific version | `nvm uninstall 16.20.2`

If there are any other special requirements, you can check them on nvm's official GitHub: [nvm](https://github.com/nvm-sh/nvm)

### Introduction to npm

npm is a package management tool for Node.js, used to manage node dependency packages in projects.

In actual project development, we use npm to install some dependencies, which can take up a lot of disk space. Below are the main cleanup items:

```bash
##### First item, node_modules #####
# Run in the corresponding root directory to view space usage
du -sh node_modules

# Cleanup method is simple (clean as needed)
rm -r node_modules

##### Second item, global dependencies #####
# Query the location of global dependencies
npm root -g

# View the size of global usage
du -sh $(npm root -g)

# Cleanup method is simple (clean as needed)
rm -r module_name

##### Third item, download cache #####
# View cache location
npm config get cache

# View cache size
du -sh $(npm config get cache)

# Clear cache
npm cache clean --force
```

### Introduction to pnpm

`pnpm` is another Node.js package manager, similar to `npm` and `yarn`, but it is more efficient in terms of performance, disk space usage, and dependency management.

Feature | Description
----- | -----
✅ Saves disk space | Uses **content-addressable** storage, placing dependencies in shared storage to avoid redundant installations
⚡ Faster installation | Dependency soft links, no copying, reducing IO operations
✅ Ensures dependency isolation | Uses strict `node_modules` structure to prevent implicit dependencies
✅ Compatible with npm/yarn | Supports most npm/yarn scripts and commands

Common command table, including comparison with npm commands:

Operation | `npm` command | `pnpm` alternative
----- | ----- | -----
Initialize project | `npm init` | `pnpm init`
Install dependencies | `npm install` | `pnpm install`
Add dependency | `npm install axios` | `pnpm add axios`
Remove dependency | `npm uninstall foo` | `pnpm remove foo`
Global install | `npm install -g` | `pnpm add -g`
Clear cache | `npm cache clean` | `pnpm store prune`

Compared to npm, pnpm manages space more cleanly. Below are some key directories:

Location | Purpose
----- | -----
`node_modules/` | Virtual dependency tree constructed using hard links
`~/.pnpm-store/` | Actual storage location for all dependency packages

Since pnpm does not store actual files in `node_modules`, there is no distinction between global and local caches. A single command is all you need to clean up all removable dependencies:

```bash
# Clean the cache
pnpm store prune
```

pnpm scans all `.pnpm-lock.yaml` files in the projects you've used, identifies unused versions, and cleans up the corresponding cache. If needed, this command can be directly added to [Auto Clean on Boot]({{< relref "#auto-clean-on-boot" >}}).

## Miscellaneous  

This section includes some simple yet commonly used commands.  

### System Control  

- Shut down immediately: `shutdown now` or `systemctl poweroff`

- Restart immediately: `sudo reboot` or `systemctl reboot`

Generally speaking, commands using `systemctl` are more "graceful," allowing running processes to end autonomously before shutdown, rather than being forcibly terminated.

Here are two recommended aliases for shutdown and reboot:

```bash
alias seeu='systemctl poweroff'
alias again='systemctl reboot'
```

### Extracting Files  

The command varies depending on the file format you need to extract.  

```bash
# Extract a .zip file  
unzip file.zip -d /target/directory  

# Extract a .tar file  
tar -xvf file.tar  

# Extract a .tar.gz file  
tar -xzvf file.tar.gz  
```

### Temporary Environment Variables

Sometimes, even with a VPN proxy enabled, the terminal may still be unable to access the internet properly. This is likely due to the terminal being unable to automatically detect the proxy port. The solution is to modify the environment variables, which can be changed permanently (by writing to a file) or temporarily, as shown below:

```shell
# For use in the PowerShell terminal on Windows
$env:HTTPS_PROXY="http://127.0.0.1:7890"

# For use in bash or cmd terminals
set HTTPS_PROXY=http://127.0.0.1:7890
```

### Temporary File Inspection

For regular files, you can just open them with the corresponding editor. However, for some special files, such as extremely large CSV files, you may only want to quickly check a small portion of the data. In this case, you can use the `head` command:

```bash
# Read and display the first 10 lines of your_file (without reading the entire file)
head -n 10 your_file.csv
```

If garbled text appears, it must be due to an encoding format issue. You can use `iconv` for conversion:

```bash
# Convert from GBK to UTF-8
head -n 10 "your_file.csv" | iconv -f gbk -t utf-8
```

If you're unsure about the original and target encodings, just throw the garbled text to an AI — it's simple and convenient 😋

## References

- [How to Partition and Mount Disks in Ubuntu](https://cloud.tencent.com/developer/article/2456171)
- [LibreOffice Suite](https://cn.linux-terminal.com/?p=1602)
- [在 Ubuntu 安装配置 Fcitx 5 中文输入法](https://muzing.top/posts/3fc249cf/)
- [Configuring Git to Push by Default Without Entering Credentials (Ubuntu, SSH)](https://blog.csdn.net/qq_22841387/article/details/145183746)
