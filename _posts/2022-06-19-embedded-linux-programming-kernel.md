---
title: "Embedded Linux Programming: Kernel"
date: 2022-06-19
categories: [Engineering,Embedded]
tags: [embedded,kernel,rpi3,rpi4]
---

# Introduction

![build pipeline]({{site.utl}}/assets/img/posts/EmbeddedProgramming2.jpg)

The kernel has three main jobs:

* to manage resources;
* to interface with hardware;
* provide an useful level of abstraction to user space program.

Applications running in __user space__ run at a low CPU privilege level. They can do very little other than make library calls. The primary interface between the __user space__ and the __kernel space__ is the __C library__, which translates user-level functions (POSIX defined) into kernel system calls. The system call interface uses an architecture-specific method such as a trap or a software interrupt, to switch the CPU from low-privilege user mode to high-privilege kernel mode, which allows access to all memory addresses and CPU registers.

The __system call handler__ dispatches the call to the appropriate kernel subsystem: memory allocation calls go to the memory manager, filesystem calls to the filesystem code, and so on.

# Prepare

Building of Linux kernel needs cross-toolchain for target platform. To get this toolchain you can use one of the following options:

* build own custom toolchain following [this](https://karz0n.github.io/posts/bake-crosstool-ng/) article;
* download any available [Linaro](https://developer.arm.com/downloads/-/gnu-a) x86_64 Linux cross toolchain.

# Build

## Build for Raspberry Pi 3/4 (ARM64)

First we set mandatory environment variables:

```shell
$ export PATH=$HOME/toolchains/aarch64-rpi3-linux-gnu/bin:$PATH
$ export CROSS_COMPILE=aarch64-rpi3-linux-gnu-
$ export ARCH=arm64
```

Build kernel:

```shell
$ sudo apt install subversion libssl-dev
$ git clone --depth=1 -b rpi-5.15.y https://github.com/raspberrypi/linux.git linux-rpi
$ pushd linux-rpi
$ make mrproper
$ make bcmrpi3_defconfig # For Raspberry Pi 3
or
$ make bcm2711_defconfig # For Raspberry Pi 4
$ make -j$(nproc)
$ popd
```

Compose files:

```shell
$ svn export https://github.com/raspberrypi/firmware/trunk/boot
$ rm boot/kernel*
$ rm boot/*.dtb
$ rm boot/overlays/*.dtbo
$ cp linux-rpi/arch/arm64/boot/Image boot/kernel8.img
$ cp linux-rpi/arch/arm64/boot/dts/overlays/*.dtbo boot/overlays
$ cp linux-rpi/arch/arm64/boot/dts/broadcom/*.dtb boot
$ cat << EOF > boot/config.txt
enable_uart=1
arm_64bit=1
EOF
$ cat << EOF > boot/cmdline.txt
console=serial0,115200 console=tty1 root=/dev/mmcblk0p2 rootwait
EOF
```

## Build for QEMU

First we set mandatory environment variables:

```shell
$ export PATH=$HOME/toolchains/arm-unknown-linux-gnueabi/bin:$PATH
$ export CROSS_COMPILE=arm-unknown-linux-gnueabi-
$ export ARCH=arm
```

Build kernel:

```sh
$ cd $HOME/embedded
$ wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.19.1.tar.xz
$ tar xf linux-5.19.1.tar.xz && rm linux-5.19.1.tar.xz
$ mv linux-5.19.1 linux-stable
$ pushd linux-stable
$ make mrproper
$ make versatile_defconfig
$ make menuconfig
# Enable options:
#  * CONFIG_BLK_DEV_INITRD
#  * CONFIG_RD_GZIP
$ make -j$(nproc) zImage
$ make -j$(nproc) modules
$ make -j$(nproc) dtbs
$ popd
```

Compose files:

```shell
$ mkdir boot
$ cp linux-stable/arch/arm/boot/zImage boot
$ cp linux-stable/arch/arm/boot/dts/*.dtb boot
$ tree boot
boot
├── versatile-ab.dtb
├── versatile-ab-ib2.dtb
├── versatile-pb.dtb
└── zImage
```

# Boot

```sh
$ sudo apt update
$ sudo apt install qemu-system-arm
$ QEMU_AUDIO_DRV=none qemu-system-arm \
-m 256M \
-nographic \
-M versatilepb \
-kernel boot/zImage \
-append "console=ttyAMA0,115200" \
-dtb boot/versatile-pb.dtb
...
[ 1.886379] Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0)
[ 1.895105] ---[ end Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0, 0)
...
```

Due to the absence of the root file system the kernel panic is a normal situation.

Note: To exit from QEMU press `Ctrl-A` + `X`.

# Command line

The kernel command line is a string that is passed to the kernel by the bootloader, via the bootargs variable in the case of U-Boot. Full documentation is available at Documentation/admin-guide/kernel-parameters.txt file.
