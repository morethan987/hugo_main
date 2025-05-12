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
series_order: 7
date: 2025-05-01
lastmod: 2025-05-01
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
# Install software  
sudo apt install xxx  

# Update packages  
sudo apt update # Sync package info from remote repositories without upgrading  
apt list --upgradable # View upgradable packages  
# Upgrade all available packages without handling dependency changes  
sudo apt upgrade  
sudo apt full-upgrade # Full upgrade  
sudo do-release-upgrade # Upgrade across major Ubuntu versions  

# check packages
sudo apt-cache search wps # check packages with key word wps

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

# Clear thumbnails  
sudo du -sh ~/.cache/thumbnails # Check thumbnail storage usage  
sudo rm -rf ~/.cache/thumbnails/* # Safe to delete—will rebuild automatically  

# Clean up old Snap versions  
snap list --all # List all Snap packages  
# List all disabled packages (single-line command)  
echo -e "\033[1 mDisabled Snap Packages and Their Sizes:\033[0 m" && snap list --all | awk '/disabled|已禁用/{print {{< katex >}}\\(1}' | while read -r pkg; do size=\\)(snap info "{{< katex >}}\\(pkg" | awk '/installed:/ {print\\)4}'); printf "%-30 s %10 s\n" "{{< katex >}}\\(pkg" "\\)size"; done | sort -k 2 -h && echo -e "\n\033[1 mTotal Size: {{< katex >}}\\((snap list --all | awk '/disabled|已禁用/{print\\)1}' | xargs -I{} snap info {} | awk '/installed:/ {sum += {{< katex >}}\\(3} END {print sum/1024 "MB"}')\\033[0 m" # Remove all disabled Snap packages (single-line command) snap list --all | awk '/disabled|已禁用/{print\\)1, {{< katex >}}\\(3}' | while read snapname revision; do snap remove "\\)snapname" --revision="$revision"; done  

# Clean up old kernels  
sudo dpkg --list | grep linux-image # List all kernels  
sudo apt autoremove --purge # Automatically remove unnecessary kernels  
```  

## References  

- [How to Partition and Mount Disks in Ubuntu](https://cloud.tencent.com/developer/article/2456171)
- [LibreOffice Suite](https://cn.linux-terminal.com/?p=1602)
