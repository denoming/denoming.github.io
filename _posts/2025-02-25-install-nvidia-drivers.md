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

Check if UEFI Secure Boot Dsiabled
```shell
$ mokutil --sb-state
SecureBoot disabled
```

Download NVIDIA driver: https://www.nvidia.com/en-us/drivers/
```shell
$ chmod +x ~/Downloads/NVIDIA-Linux-x86_64-550.144.03.run
```

Remove NVIDIA drivers
```shell
$ sudo dpkg -l "*nvidia*" |grep ii |awk '{print $2}'
$ sudo apt remove $(dpkg -l "*nvidia*"" |grep ii |awk '{print $2}')
$ sudo apt reinstall xserver-xorg-video-nouveau
$ reboot
```

```shell
$ pkcon update
$ pkcon install linux-headers-$(uname -r) gcc gcc-12 g++-12 make acpid dkms pkg-config libglvnd-core-dev libglvnd0 libglvnd-dev libc-dev xwayland libxcb1
```

```shell
$ update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-12 90 --slave /usr/bin/g++ g++ /usr/bin/g++-12
$ update-alternatives --config gcc
```

```shell
$ wget https://mirrors.edge.kernel.org/ubuntu/pool/main/e/egl-wayland/libnvidia-egl-wayland1_1.1.17-1_amd64.deb
$ sudo dpkg -i libnvidia-egl-wayland1_1.1.17-1_amd64.deb
```

```shell
$ sudo -i
$ echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf
$ echo "options nvidia_drm modeset=1" >> /etc/modprobe.d/nvidia.conf
$ echo "options nvidia_drm fbdev=1" >> /etc/modprobe.d/nvidia.conf
```

```shell
$ update-grub2
$ update-initramfs -c -k $(uname -r)
```

```shell
$ systemctl set-default multi-user.target
$ reboot
```

```shell
sudo -i
~/Downloads/NVIDIA-Linux-*.run
```

> Select NVIDIA Propriatary
> Select Continue Installation
> Yes
> Yes
> Rebuild initramfs
> Yes

```shell
systemctl set-default graphical.target
reboot
```

```shell
$ pkcon install vdpauinfo libva2 vainfo
$ sudo systemctl enable nvidia-suspend.service
$ sudo systemctl enable nvidia-hibernate.service
$ sudo systemctl enable nvidia-resume.service

$ nvidia-installer -v | grep version
nvidia-installer:  version 550.144.03
```
