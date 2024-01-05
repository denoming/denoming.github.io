---
title: "Embedded Linux Programming: Root Filesystem"
date: 2022-06-19
categories: [Engineering,Embedded]
tags: [embedded,toolchain]
---

# Prepare

## Create folders layout

```sh
$ mkdir $HOME/embedded/rootfs
$ cd $HOME/embedded/rootfs
$ mkdir bin dev etc home lib proc sbin sys tmp usr var
$ mkdir usr/bin usr/lib usr/sbin var/log
$ tree -d
.
├── bin
├── dev
├── etc
├── home
├── lib
├── proc
├── sbin
├── sys
├── tmp
├── usr
│   ├── bin
│   ├── lib
│   └── sbin
└── var
    └── log
```

## Install core utils

```sh
$ cd $HOME/embedded
$ source $HOME/embedded/toolchain/environment.arm
$ git clone git://busybox.net/busybox.git
$ cd busybox
$ git switch 1_35_stable
$ make distclean
$ make defconfig
$ make menuconfig
# Specify /home/<user>/embedded/rootfs as installation prefix in "Settings | Installation Optionas (CONFIG_PREFIX)" option
$ make -j$(nproc)
$ make install
```

## Install libraries

Recognize the set of necessary libraries:

```sh
$ source $HOME/embedded/toolchain/environment.arm
$ cd $HOME/embedded/rootfs
$ export SYSROOT=$(arm-unknown-linux-gnueabi-gcc -print-sysroot)
$ arm-unknown-linux-gnueabi-readelf -a bin/busybox | grep "program interpreter"
      [Requesting program interpreter: /lib/ld-linux.so.3]
$ arm-unknown-linux-gnueabi-readelf -a bin/busybox | grep "Shared library"
 0x00000001 (NEEDED)                     Shared library: [libm.so.6]
 0x00000001 (NEEDED)                     Shared library: [libresolv.so.2]
 0x00000001 (NEEDED)                     Shared library: [libc.so.6]
$ cd $SYSROOT
$ ls -l lib/ld-linux.so.3
-rwxr-xr-x 1 denys denys 1231644 May 13 21:25 lib/ld-linux.so.3
$ ls -l lib/libm.so.6
-rwxr-xr-x 1 denys denys 1803108 May 13 21:25 lib/libm.so.6
$ ls -l lib/libresolv.so.2
-rwxr-xr-x 1 denys denys 247984 May 13 21:25 lib/libresolv.so.2
$ ls -l lib/libc.so.6
-rwxr-xr-x 1 denys denys 12375828 May 13 21:25 lib/libc.so.6
```

Copy the set of necessary libraries:

```sh
$ cd $HOME/embedded/rootfs
$ cp -a $SYSROOT/lib/ld-linux.so.3 lib
$ cp -a $SYSROOT/lib/libm.so.6 lib
$ cp -a $SYSROOT/lib/libresolv.so.2 lib
$ cp -a $SYSROOT/lib/libc.so.6 lib
```

### Strip libraries (optional)

```sh
$ arm-unknown-linux-gnueabi-strip lib/libc.so.6
# To strip kernel modules use *-strip --strip-unneeded <module name> command
```

## Create device nodes

```sh
$ cd $HOME/embedded/rootfs
$ sudo mknod -m 666 dev/null c 1 3
$ sudo mknod -m 600 dev/console c 5 1
$ ls -l dev
```

Note: All major/minor number pairs for mknod are accessible in Documentation/admin-guide/devices.txt file

# Creating

## Standalone initramfs

```sh
$ tree $HOME/embedded/boot
boot
├── versatile-ab.dtb
├── versatile-ab-ib2.dtb
├── versatile-pb.dtb
└── zImage
$ cd $HOME/embedded
$ pushd rootfs # It's mandatory to be located in the rootfs dir
$ find . | cpio -H newc -ov --owner root:root > ../boot/initramfs.cpio
$ popd
$ gzip -fk boot/initramfs.cpio
$ mkimage -A arm -O linux -T ramdisk -d boot/initramfs.cpio.gz boot/uRamdisk
```

Note: Files in boot folder is created following "Embedded Linux Programming: Kernel" article.

# Booting

## QEMU
Boot in QEMU using available `initrd` flag:

```shell
$ sudo apt update
$ sudo apt install qemu-system-arm
$ QEMU_AUDIO_DRV=none qemu-system-arm \
-m 256M \
-nographic \
-M versatilepb \
-kernel boot/zImage \
-append "root=/dev/ram0 console=ttyAMA0 rdinit=/bin/sh" \
-dtb boot/versatile-pb.dtb \
-initrd boot/initramfs.cpio.gz
...
/ #
```

After kernel boots you will be able to see a shell prompt.

Note: To exit from QEMU press `Ctrl-A` + `X`.

## Embed initramfs into the kernel image (optional)

There are bootloaders that not support loading an initramfs file. Therefore, we can embed
initramfs into kernel image. For that purpose, there is an option in menu-config that
gives such possibility.

```shell
$ cd $HOME/embedded/linux-stable
$ pushd linux-stable
$ make mrproper
$ make versatile_defconfig
$ make menuconfig
# Go to "General setup | Initramfs source file(s)"
# and change to $HOME/embedded/boot/initramfs.cpio
$ make -j$(nproc) zImage
$ make -j$(nproc) modules
$ make -j$(nproc) dtbs
$ popd
$ cp linux-stable/arch/arm/boot/zImage boot/zImage-embed
$ ls -lh arch/arm/boot/zImage*
-rwxrwxr-x 1 denys denys 3,2M Aug 11 22:25 boot/zImage
-rwxrwxr-x 1 denys denys 5,7M Aug 11 23:03 boot/zImage-embed
```

Booting:
```shell
$ QEMU_AUDIO_DRV=none qemu-system-arm \
-m 256M \
-nographic \
-M versatilepb \
-kernel boot/zImage-embed \
-append "console=ttyAMA0 rdinit=/bin/sh" \
-dtb boot/versatile-pb.dtb
...
/ #
```

Note: To exit from QEMU press `Ctrl-A` + `X`.

## The init program

### Basic init program
The BusyBox `init` program begins by reading the configuration file, __/etc/inittab__. Here is a simple example of this config file and running shell script.
```shell
$ cd $HOME/embedded/rootfs
$ tee etc/inittab > /dev/null <<EOF
::sysinit:/etc/init.d/rcS
::askfirst:-/bin/ash
EOF
$ mkdir etc/init.d
$ tee etc/init.d/rcS > /dev/null <<EOF
#!/bin/sh
mount -t proc proc /proc
mount -t sysfs sysfs /sys
EOF
$ chmod +x etc/init.d/rcS
```

The first line of `inittab` runs a shell script `rcS`. The second line prints the prompt message and starts a shell when you press _Enter_. The leading symbol `-` before `/bin/ash` means that it will become a login shell.
The `rcS` scipr is the place to put initialization commands that need to be performed at boot (e.g. mounting the `proc` and `sysfs` filesystems).

Update initramfs file following "Creating => Standalone initramfs". After updating run QEMU with specifying `initrd` argument and `rdinit` kernel option:
```shell
$ QEMU_AUDIO_DRV=none qemu-system-arm \
-m 256M \
-nographic \
-M versatilepb \
-kernel boot/zImage \
-append "console=ttyAMA0 rdinit=/sbin/init" \
-dtb boot/versatile-pb.dtb \
-initrd boot/initramfs.cpio.gz
```
The above QEMU command line is a base way to run kernel with initramfs as a root filesystem and `init` as a start program.

###  Add starting a daemon

To do this we need to update `etc/inittab`:
```shell
$ cd $HOME/embedded/rootfs
$ tee -a etc/inittab > /dev/null <<EOF
::respawn:/sbin/syslogd -n
::respawn:/sbin/klogd -n
EOF
```

Update initramfs file following "Creating => Standalone initramfs". Next, run QEMU as in previous chapter and observe a presens of `syslogd` and `klogd` daemons:
```text
/ # ps -a
...
   33 0         0:00 /sbin/syslogd -n
   34 0         0:00 /sbin/klogd -n
```

### Add user accounts

Add two user accounts for `root` and `daemon` with particular group and records in `etc/shadow` file:
```shell
$ cd $HOME/embedded/rootfs
$ tee etc/passwd > /dev/null <<EOF
root:x:0:0:root:/root:/bin/sh
daemon:x:1:1:daemon:/usr/sbin:/bin/false
EOF
$ tee etc/shadow > /dev/null <<EOF
root::10933:0:99999:7:::
daemon:*:10933:0:99999:7:::
EOF
$ tee etc/group > /deb/null <<EOF
root:x:0:
daemon:x:1:
EOF
```

Update `etc/inittab` file to spawn `getty` that will request an user to login:
```shell
$ tee etc/inittab > /deb/null <<EOF
::sysinit:/etc/init.d/rcS
::respawn:/sbin/syslogd -n
::respawn:/sbin/klogd -n
::respawn:/sbin/getty 115200 console
EOF
```

Finally, update initramfs file and run QEMU as usual.

### Manager device nodes

First of wall we need enable devtmpfs supporting on kernel level. Therefore, following instructions in _Embedded Linux Programming: Kernel_ article we need update our kernel but enable `CONFIG_DEVTMPFS` kernel option beforehand. After this manipulations we are able to mount `devtmpfs` upon `/dev` path.

Check init config files:
```shell
$ cd $HOME/embedded/rootfs
$ tee etc/inittab > /dev/null <<EOF
::sysinit:/etc/init.d/rcS
::respawn:/sbin/syslogd -n
::respawn:/sbin/klogd -n
::askfirst:-/bin/ash
EOF
$ tee etc/init.d/rcS > /dev/null <<EOF
mount -t proc proc /proc
mount -t sysfs sysfs /sys
mount -t devtmpfs devtmpfs /dev
echo /sbin/mdev > /proc/sys/kernel/hotplug
mdev -s
EOF
$ tee etc/mdev.conf > /dev/null <<EOF
null root:root 666
random root:root 444
urandom root:root 444
EOF
```

Update initramfs file following "Creating => Standalone initramfs". Next, run QEMU as usual and you will see a lot more devices at `/dev` location:
```text
/ # ls /dev
console    ptyp3      tty        tty25      tty42      tty6       ttyp7
dri        ptyp4      tty0       tty26      tty43      tty60      ttyp8
fb0        ptyp5      tty1       tty27      tty44      tty61      ttyp9
full       ptyp6      tty10      tty28      tty45      tty62      ttypa
gpiochip0  ptyp7      tty11      tty29      tty46      tty63      ttypb
gpiochip1  ptyp8      tty12      tty3       tty47      tty7       ttypc
gpiochip2  ptyp9      tty13      tty30      tty48      tty8       ttypd
gpiochip3  ptypa      tty14      tty31      tty49      tty9       ttype
kmsg       ptypb      tty15      tty32      tty5       ttyAMA0    ttypf
log        ptypc      tty16      tty33      tty50      ttyAMA1    urandom
mem        ptypd      tty17      tty34      tty51      ttyAMA2    vcs
mtd0       ptype      tty18      tty35      tty52      ttyAMA3    vcs1
mtd0ro     ptypf      tty19      tty36      tty53      ttyp0      vcsa
mtdblock0  ram0       tty2       tty37      tty54      ttyp1      vcsa1
null       ram1       tty20      tty38      tty55      ttyp2      vcsu
ptmx       ram2       tty21      tty39      tty56      ttyp3      vcsu1
ptyp0      ram3       tty22      tty4       tty57      ttyp4      zero
ptyp1      random     tty23      tty40      tty58      ttyp5
ptyp2      rtc0       tty24      tty41      tty59      ttyp6
```

Configuring `mdev` by creating `etc/mdev.conf` config file we are able to update permissions on device nodes. Additionally you are able to do a lot of more things than just change permissions of device nodes.

### Configure network

```shell
$ cd $HOME/embedded/rootfs
$ mkdir -p etc/network/{if-pre-up.d,if-up.d}
$ mkdir -p var/run
```

Static IP configuration:
```shell
$ tee etc/network/interfaces > /dev/null <<EOF
auto lo
iface lo inet loopback
auto eth0
iface eth0 inet static
    address 192.168.1.101
    netmask 255.255.255.0
    network 192.168.1.0
EOF
```

Dynamic IP configuration:
```shell
$ tee etc/network/interfaces > /dev/null <<EOF
auto lo
iface lo inet loopback
auto eth0
iface eth0 inet dhcp
EOF
$ mkdir -p usr/share/udhcpc
$ tee usr/share/udhcpc/default.script > /dev/null <<EOF

EOF
```


```shell
$ QEMU_AUDIO_DRV=none qemu-system-arm \
-m 256M \
-nographic \
-M versatilepb \
-kernel boot/zImage \
-net nic -net tap,ifname=tap0,script=no \
-append "console=ttyAMA0 rdinit=/sbin/init" \
-dtb boot/versatile-pb.dtb \
-initrd boot/initramfs.cpio.gz
```


```shell
$ sudo apt install bridge-utils,uml-utilities
$ ifconfig
...
enp0s31f6: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.20  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::4e66:3a5b:cb25:5b5e  prefixlen 64  scopeid 0x20<link>
        ether 9c:5c:8e:74:28:4f  txqueuelen 1000  (Ethernet)
...
$ sudo su
% brctl addbr br0
% ip addr flush dev enp0s31f6
% brctl addif br0 enp0s31f6
% tunctl -t tap0 -u `whoami`
% brctl addif br0 tap0
% ifconfig enp0s31f6 up
% ifconfig tap0 up
% ifconfig br0 up
% brctl show
bridge name	bridge id		STP enabled	interfaces
br0		8000.e218993651c7	no		enp0s31f6
							tap0
% dhclient -v br0
```