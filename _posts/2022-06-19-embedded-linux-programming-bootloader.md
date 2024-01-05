---
title: "Embedded Linux Programming: Bootloader"
date: 2022-06-19
categories: [Engineering,Embedded]
tags: [embedded,bootloader,u-boot]
---

# Introduction

![bootloader in action]({{site.utl}}/assets/img/posts/EmbeddedProgramming3.svg)

Reset vector points out to vendor specific ROM code. After reet or power-on ROM code is run. Due to limitation of memory the only responsibility of ROM code is to load SPL code into SRAM (Static Random Access Memory) memory and jumps to it.
SPL code is responsible to set up memory controller and other essential code. At the end is loads TPL (tertiary program loader) into DRAM and jumps to it.
TPL (e.g. Uboot) represents a full bootloader. When TPL executes it loads the kernel into DRAM. When the bootloader passes control to the kernel, it has to pass basic information, which includes:
* the machine number (when device tree is not supported);
* basic details of the hardware (the size and location of the physicall RAM and the CPU's clock speed);
* the kernel command line;
* the location and size of a device tree binary (optionally);
* the location and size of an initial RAM disk (optionally).

The way information is passes is depended on the architecture. The universal way is a device tree. Usin device brings a possibility to have the same kernel binary for multiple platforms.

# Prepare

Building of U-Boot needs cross-toolchain for target platform. To get this toolchain you can use one of the following options:

* build own custom toolchain following [this](https://karz0n.github.io/posts/bake-crosstool-ng/) article;
* download any available [Linaro](https://developer.arm.com/downloads/-/gnu-a) x86_64 Linux cross toolchain.

# Build

As example, we are going to build U-Boot for ARM architecture with target Raspberry Pi 3 board. We expect that toolchain is located at `$HOME/toolchains/arm-unknown-linux-gnueabi` folder.

First we set mandatory environment variables:

```shell
$ export PATH=$HOME/toolchains/arm-unknown-linux-gnueabi/bin
$ export CROSS_COMPILE=arm-unknown-linux-gnueabi-
$ export ARCH=arm
```

Build U-Boot from source:

```shell
$ git clone --depth 1 --branch v2022.04 git://git.denx.de/u-boot.git
$ pushd u-boot
$ make rpi_3_defconfig
$ make -j$(nproc)
$ popd
```

Download additional files:

```shell
$ mkdir firmware && pushd firmware
$ wget \
https://github.com/raspberrypi/firmware/raw/master/boot/bootcode.bin \
https://github.com/raspberrypi/firmware/raw/master/boot/fixup.dat \
https://github.com/raspberrypi/firmware/raw/master/boot/start.elf
$ popd
```

Create configuration file:

```shell
$ tee > /dev/null /media/$USER/boot/config.txt <<EOF
enable_uart=1
kernel=kernel8.bin
arm_64bit=1
core_freq=250
EOF
```

# Upload

Prepare partitions on microSD card:

```shell
$ lsblk
...
sde      8:64   1  58,2G  0 disk
├─sde1   8:65   1  48,4M  0 part /media/.../boot
└─sde2   8:66   1 109,2M  0 part /media/.../root
$ sudo umount /dev/sde[1-2]
$ sudo dd if=/dev/zero of=/dev/sde bs=1M count=10
$ sudo sfdisk /dev/sde << EOF
,128M,0x0c,*
,4096M,L,
EOF
$ sudo mkfs.vfat -F 16 -n boot /dev/sde1
$ sudo mkfs.ext4 -L root /dev/sde2
```

Re-insert microSD card to mount all partitions.

Upload files on microSD card:

```shell
$ cp firmware/{bootcode.bin,fixup.dat,start.elf} /media/$USER/boot
$ cp u-boot/u-boot.bin /media/$USER/boot/kernel8.bin
$ sync
```

Insert microSD card to the board and power up it.
