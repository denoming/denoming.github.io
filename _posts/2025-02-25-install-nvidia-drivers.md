---
title: Install NVIDIA drviers
date: 2025-02-25
categories:
  - Linux
tags:
  - video
---
Check if NVIDIA card supported:
```shell
$ lspci |grep -E "VGA|3D"
01:00.0 VGA compatible controller: NVIDIA Corporation GA102 [GeForce RTX 3080 Lite Hash Rate] (rev a1)
```

Check if UEFI Secure Boot Disabled:
```shell
$ mokutil --sb-state
SecureBoot disabled
```

# Debian

## Install NVIDIA driver (Debian way)

See [Debian 13 "Trixie"](https://wiki.debian.org/NvidiaGraphicsDrivers#Debian_13_.22Trixie.22).

## Install NVIDIA CUDA driver + toolkit (origin)

Enroll public keys and configure NVIDIA repository:
```shell
$ wget https://developer.download.nvidia.com/compute/cuda/repos/debian13/x86_64/cuda-keyring_1.1-1_all.deb
$ sudo dpkg -i cuda-keyring_1.1-1_all.deb
$ sudo apt-get update
```

Install proprietary kernel module (preferably to run in recovery mode):
```shell
sudo apt-get install -y cuda-drivers
```

Enable DRM in GRUB configuration mode:
```shell
$ sudo echo 'GRUB_CMDLINE_LINUX="$GRUB_CMDLINE_LINUX nvidia-drm.modeset=1 nvidia-drm.fbdev=1"' > /etc/default/grub.d/nvidia-modeset.cfg
$ sudo update-grub
```

Force loading the driver on early stages:
```shell
sudo vim /etc/initramfs-tools/modules
...
nvidia
nvidia_modeset
nvidia_uvm
nvidia_drm
```

Enable NVIDIA module parameter:
```shell
$ sudo echo 'options nvidia NVreg_PreserveVideoMemoryAllocations=1' > /etc/modprobe.d/nvidia-power-management.conf
```
Reboot:
```shell
$ sudo systemctl restart
```

After restarting install CUDA toolkit:
```shell
$ sudo apt-get -y install cuda-toolkit-13-1
```