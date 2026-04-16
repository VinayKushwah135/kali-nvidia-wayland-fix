# 🐉 Kali Linux NVIDIA + Wayland Survival Guide

![Kali Linux](https://img.shields.io/badge/Kali_Linux-Rolling-blue?style=flat-square&logo=kalilinux)
![NVIDIA](https://img.shields.io/badge/NVIDIA-550%2B-76B900?style=flat-square&logo=nvidia)
![Bash](https://img.shields.io/badge/Script-Bash-4EAA25?style=flat-square&logo=gnu-bash)
![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)
![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)

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


1. Nuke the Broken State
Clear out any interrupted package configurations and purge all existing NVIDIA packages:

Bash
sudo dpkg --configure -a
sudo apt purge "^nvidia-.*" "^libnvidia-.*"
sudo apt autoremove --purge
sudo apt clean
2. Update Sources & Install Headers
Ensure you are on the kali-rolling branch, then update and grab the headers for your specific kernel:

Bash
sudo apt update && sudo apt full-upgrade -y
sudo apt install linux-headers-$(uname -r)
3. Install the Driver
Install the driver, the X configuration tool, and the essential GSP firmware:

Bash
sudo apt install nvidia-driver firmware-nvidia-gsp nvidia-xconfig
4. Fix Wayland (DRM Modesetting)
Wayland requires nvidia-drm.modeset=1. Add it to GRUB:

Bash
# 1. Edit GRUB
sudo nano /etc/default/grub
# Change GRUB_CMDLINE_LINUX_DEFAULT to include "nvidia-drm.modeset=1"

# 2. Update GRUB
sudo update-grub

# 3. Force the module into early boot
echo "nvidia-drm" | sudo tee -a /etc/initramfs-tools/modules
sudo update-initramfs -u

# 4. Reboot your system
sudo reboot
💻 Usage & Verification
Once rebooted, verify the installation:

Check the GPU:

Bash
nvidia-smi
You should see your GPU listed with Driver Version 550+.

Check DRM Modesetting:

Bash
cat /sys/module/nvidia_drm/parameters/modeset
This must output Y.

Verify Wayland Session:
Log into a Wayland session (GNOME recommended) and run:

Bash
echo $XDG_SESSION_TYPE
This should output wayland.









