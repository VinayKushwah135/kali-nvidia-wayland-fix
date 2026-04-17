# 🐉 Kali Linux NVIDIA + Wayland Survival Guide

![Kali Linux](https://img.shields.io/badge/Kali_Linux-Rolling-blue?style=flat-square&logo=kalilinux)
![NVIDIA](https://img.shields.io/badge/NVIDIA-550%2B-76B900?style=flat-square&logo=nvidia)
![Bash](https://img.shields.io/badge/Script-Bash-4EAA25?style=flat-square&logo=gnu-bash)


If you've ever found yourself stuck in DKMS dependency hell, staring at `bad exit status: 2` errors, or wondering why Wayland is completely ignoring your RTX card on Kali Linux — you're in the right place. 

This repository documents the exact, tested workflow to nuke broken driver states, force the Kali Rolling repositories to give you the modern 550+ driver stack, and inject the necessary DRM modesetting into GRUB and Initramfs so Wayland actually boots.

## ⚠️ The Problem Statement

Kali Linux is a rolling release, meaning its kernel updates rapidly (e.g., Kernel 6.18+). If your APT sources are stale or you try to install an older proprietary driver (like the 535 series), the DKMS build will fail because the old driver code cannot compile against the new kernel APIs. 

Furthermore, even if you successfully install the driver, Wayland will not recognize the NVIDIA GPU unless Direct Rendering Manager (DRM) modesetting is explicitly enabled in the bootloader and loaded into the initial ramdisk.

## ✨ Features

- **The "Nuclear" Cleanse:** Safely purges half-configured or broken `nvidia-*` and `libnvidia-*` packages.
- **Header Synchronization:** Ensures your Linux headers perfectly match your running kernel.
- **Modern Driver Stack:** Installs the 550+ driver series alongside the required GSP firmware.
- **Wayland/DRM Enabler:** Automates the patching of `/etc/default/grub` and `/etc/initramfs-tools/modules`.


These are steps to fix broken NVIDIA drivers and configure Wayland on Kali Linux.
It ranges from changing/editing the repository to nuking broken drivers and injecting DRM modesetting into Optimus/nvidia laptops.

## for the command procedure on terminal make sure to switch to root user , If you haven't already created the root user, please make sure to do so :
### Set the password for root
```bash
sudo passwd
```
Switch to root:
```bash
sudo su
```

 ## 1.Nuke the Broken State
  Clear out any interrupted package configurations and purge all existing NVIDIA packages:
```bash
sudo dpkg --configure -a
sudo apt purge "^nvidia-.*" "^libnvidia-.*"
sudo apt autoremove --purge
sudo apt clean
```

## 2.Update Sources & Install Headers
Ensure you are on the kali-rolling branch, then update and grab the headers for your specific kernel:
```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt install linux-headers-$(uname -r)
```
### if any errors shows in red theme about the linux headers just run this command
``` bash
─# sudo apt update                                                  
sudo apt dist-upgrade -y
```
### and then reboot it
```bash
reboot
```

## 3.Install the Driver
Install the driver, the X configuration tool, and the essential GSP firmware:
```bash
sudo apt install nvidia-driver firmware-nvidia-gsp nvidia-xconfig
```
## 4.Fix Wayland (DRM Modesetting)
Wayland requires nvidia-drm.modeset=1. Edit GRUB:

```bash
sudo nano /etc/default/grub
```
```bash
Change GRUB_CMDLINE_LINUX_DEFAULT to include "nvidia-drm.modeset=1"
```
## 5. update the grub
Update GRUB:
```bash
sudo update-grub
```
## 6.Force the module into early boot:
```bash
echo "nvidia-drm" | sudo tee -a /etc/initramfs-tools/modules
sudo update-initramfs -u
```
## 7.Reboot your system:
```bash
sudo reboot
```
# Usage & Verification
Once rebooted, verify the installation.
Check the GPU:
```bash
nvidia-smi
```
###Check DRM Modesetting:
```bash
cat /sys/module/nvidia_drm/parameters/modeset
```
This must output Y.

### Verify Wayland Session:
```bash
echo $XDG_SESSION_TYPE
```
This should output wayland.

## if you are on Kali's default XFCE desktop (which doesn't support Wayland well), you have two choices to actually see Wayland in action:

### Option A: The GNOME Way (Most Stable)

GNOME is the "gold standard" for NVIDIA + Wayland.

Install GNOME:
```bash
sudo apt install kali-desktop-gnome
```

Reboot or Log Out.

Switch Session: At the login screen, click your name, then click the Gear Icon in the bottom-right corner. Select "GNOME on Wayland"

### Option B: The KDE Way (Most Customizable)
Install KDE: 
```bash
sudo apt install kali-desktop-kde
```
log out or reboot the system

Switch Session: Select "Plasma (Wayland)" from the session menu.








