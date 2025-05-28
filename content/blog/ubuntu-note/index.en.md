---
title: Ubuntu Tinkering Notes
weight: -95
draft: false
description: Documenting the process of tinkering with Ubuntu for future reference
slug: ubuntu-note
tags:
  - Ubuntu
  - Linux
series:
  - Technical Miscellany
series_order: 8
date: 2025-05-01
lastmod: 2025-05-20
authors:
  - Morethan
---
{{< lead >}}
Summarizing and documenting the process of tinkering with Ubuntu for future reference.  
{{< /lead >}}

## Introduction  

Ubuntu, as a popular Linux distribution, offers better ecosystem support compared to other Linux distros. The most notable advantage is that when you encounter issues, you're more likely to find tutorials and solutions for Ubuntu.  

This article primarily focuses on the GUI-based personal edition of Ubuntu. For server-specific Ubuntu systems, operations depend on your actual business needs. The article [Cloud Server Setup]({{< ref "/blog/cloud-server-build/" >}}) can serve as a reference.  

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

## Configuring Git

One of the standout features of Linux is its extreme simplicity, which is why using the command line to manage Git is the preferred choice for Linux users 😃. Ubuntu comes with Git pre-installed, so there's no need to install it separately. If you want to upgrade, follow these steps:

```bash
git --version # Check the Git version

sudo add-apt-repository ppa:git-core/ppa # Add the official repository

sudo apt update && sudo apt upgrade # If possible, proceed with the upgrade
```

### Setting Up SSH for GitHub (Password-Free Configuration)

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
# Search for all packages containing "wps" and their description information
sudo apt-cache search wps
# View package names containing the keyword "wps"
sudo apt-cache pkgnames | grep -i wps

# Remove packages  
sudo apt remove xxx  
sudo apt autoremove # Clean up residuals  
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
# Check disk usage of folders under .cache
du -sh ~/.cache/* | sort -h -r
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

After modifying the file, execute:  

```bash  
# Refresh the package index and perform a full upgrade, including the kernel, drivers, and all packages  
sudo apt update && sudo apt full-upgrade -y  
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

## Miscellaneous  

This section includes some simple yet commonly used commands.  

### System Control  

- Shut down immediately:  

```bash
  shutdown now
```  

- Restart immediately:  

```bash
  sudo reboot
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

## References  

- [How to Partition and Mount Disks in Ubuntu](https://cloud.tencent.com/developer/article/2456171)
- [LibreOffice Suite](https://cn.linux-terminal.com/?p=1602)
- [在 Ubuntu 安装配置 Fcitx 5 中文输入法](https://muzing.top/posts/3fc249cf/)
- [Configuring Git to Push by Default Without Entering Credentials (Ubuntu, SSH)](https://blog.csdn.net/qq_22841387/article/details/145183746)
