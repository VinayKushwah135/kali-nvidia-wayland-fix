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


