---
title: "Bake yocto linux for RaspberryPi 3"
date: 2020-03-01
categories: [Engineering,Yocto]
tags: [yocto,linux,raspberrypi3]
---

# Useful Links

[Quick build](https://docs.yoctoproject.org/brief-yoctoprojectqs/index.html) 

# Install dependencies

```shell
$ sudo apt-get install -y vim gawk wget curl git git-lfs diffstat unzip texinfo \
       lz4 zstd gcc-multilib g++-multilib build-essential chrpath socat cpio \
       python3 python3-pip python3-pexpect python3-git python3-jinja2 \
       python3-subunit python-is-python3 xz-utils debianutils iputils-ping \
       pylint3 xterm mesa-common-dev libegl1-mesa libsdl1.2-dev
```

For details see: [Required packages for the build host](https://docs.yoctoproject.org/brief-yoctoprojectqs/index.html#build-host-packages).

# Download

To build raspberry-pi3 image we need to get _poky_ and two additional layers: _meta-raspberrypi_ and _meta-openembedded_:

```shell
$ mkdir -p $HOME/yocto/sources && cd yocto/sources
$ git clone -b kirkstone git://git.yoctoproject.org/poky
$ git clone -b kirkstone git://git.openembedded.org/meta-openembedded
$ git clone -b kirkstone git://git.yoctoproject.org/meta-raspberrypi
```

We checkout `kirkstone` stable branch. To get all available release branches follow next link - [Releases](https://wiki.yoctoproject.org/wiki/Releases).

# Configure

```shell
$ cd $HOME/yocto
$ source sources/poky/oe-init-build-env build-rpi3
$ bitbake-layers add-layer $PWD/../sources/meta-openembedded/meta-oe
$ bitbake-layers add-layer $PWD/../sources/meta-openembedded/meta-python
$ bitbake-layers add-layer $PWD/../sources/meta-openembedded/meta-multimedia
$ bitbake-layers add-layer $PWD/../sources/meta-openembedded/meta-networking
$ bitbake-layers add-layer $PWD/../sources/meta-raspberrypi
$ build-rpi3 bitbake-layers show-layers                                
NOTE: Starting bitbake server...
layer                 path                                      priority
==========================================================================
meta                  .../sources/poky/meta                          5
meta-poky             .../sources/poky/meta-poky                     5
meta-yocto-bsp        .../sources/poky/meta-yocto-bsp                5
meta-oe               .../sources/meta-openembedded/meta-oe          5
meta-python           .../sources/meta-openembedded/meta-python      5
meta-multimedia       .../sources/meta-openembedded/meta-multimedia  5
meta-networking       .../sources/meta-openembedded/meta-networking  5
meta-raspberrypi      .../sources/meta-raspberrypi                   9
```

Set local configuration at `conf/local.conf` file:
```text
# Machine Selection
MACHINE ?= "raspberrypi3-64"

# Default policy config
DISTRO ?= "poky"

# Install minimal package set
CORE_IMAGE_EXTRA_INSTALL = "\
	splash \
	ssh-server-openssh \
    connman \
    connman-client \
    e2fsprogs \
    e2fsprogs-e2fsck \
    e2fsprogs-badblocks \
    e2fsprogs-resize2fs \    
"

# Package Management configuration
PACKAGE_CLASSES ?= "package_ipk"

# SDK target architecture
SDKMACHINE ?= "x86_64"

# Additional image features
USER_CLASSES ?= "buildstats"

# Interactive shell configuration
PATCHRESOLVE = "noop"

# Exclude some features (save time during build)
DISTRO_FEATURES:remove = " x11 wayland vulkan 3g opengl"

# Specify license white list
LICENSE_FLAGS_ACCEPTED = "commercial synaptics-killswitch"

# Specify version for tracking
CONF_VERSION = "2"

# Specify image filesystem types
IMAGE_FSTYPES = "tar.bz2 ext4 rpi-sdimg"

# Increase free disk space
IMAGE_ROOTFS_EXTRA_SPACE = "1048576"
IMAGE_OVERHEAD_FACTOR = "1.5"

# Use systemd for system initialization
DISTRO_FEATURES:append = " systemd"
DISTRO_FEATURES_BACKFILL_CONSIDERED:append = " sysvinit"
VIRTUAL-RUNTIME_init_manager = "systemd"
VIRTUAL-RUNTIME_initscripts = "systemd-compat-units"
VIRTUAL-RUNTIME_login_manager = "shadow-base"
VIRTUAL-RUNTIME_dev_manager = "systemd"
```

# Build and flash image
```shell
$ bitbake rpi-test-image
```

Inject SD card to PC and determine which name device has:
```shell
$ lsblk
...
sdd      8:48   1    58G  0 disk 
├─sdd1   8:49   1    40M  0 part /media/denis/raspberrypi
└─sdd2   8:50   1  57,9G  0 part /media/denis/116dc20e-1fbb-4d1c-9951-f2a8ef902f11
...
```

For my case it has _sdd_ name and consists of two parts (sdd1 and sdd2). Before we continue we need to unmount these partitions:
```shell
$ sudo umount /dev/sdd1
$ sudo umount /dev/sdd2
```

Write image file on SD card::
```shell
sudo dd if=$HOME/yocto/build-rpi3/tmp/deploy/images/raspberrypi3-64/<name-of-file-image>.rpi-sdimg of=/dev/sdd bs=1M status=progress
```

# Optional: Expand filesystem on SD card to take whole space

Inject SD card and login to console on device and enter next commands:
```shell
# mount
/dev/mmcblk0p2 on / type ext4 ...
...
# fdisk -u /dev/mmcblk0
> p # print all partitions (remember the start of the second partition)
> d # delete partition
> 2 # select the second partition to delete
> n # create new partition
> p # specify new partition as primary
> 2 # specify parition number
> ~ # specify the start of partition 
> ~ # specify the end of parition
> ~ # reject to remove the signature
> p # print all partitions
> w # write changes
> q

# resize2fs /dev/mmcblk0p2
```

# Optional: Build SDK image

To build SDK image we need to specify _SDKMACHINE_ to _x86_64_ which means that produced gcc-crosssdk toolchain will run on _x86_64_ machine type build host and produce code for _MACHINE_ type target.
```shell
$ vim conf/local.conf
...
SDKMACHINE ?= "x86_64"
...
```

Now we are able to build SDK toolchain:
```shell
$ bitbake core-image-base -c populate_sdk_ext
$ cd tmp/deploy/sdk
$ ls -1
...
poky-glibc-x86_64-core-image-base-aarch64-raspberrypi3-64-toolchain-ext-2.7.1.sh
```

# Optional: Configure Wi-Fi
```shell
$ ssh root@<ip-address>
$ lsmod | grep 80211 # Check if Wi-Fi driver is on board
cfg80211              876544  1 brcmfmac
rfkill                 36864  5 bluetooth,cfg80211
$ connmanctl
connmanctl> enable wifi
Enabled wifi
connmanctl> agent on
Agent registered
connmanctl> scan wifi
Scan completed for wifi
connmanctl> services
*AR Wired                ethernet_b827ebe067f2_cable
...
ZION                 wifi_b827ebb532a7_5a494f4e_managed_psk
...
connmanctl> connect wifi_b827ebb532a7_5a494f4e_managed_psk
...
Connected wifi_b827ebb532a7_5a494f4e_managed_psk
connmanctl> services
*AR Wired                ethernet_b827ebe067f2_cable
*AR ZION                 wifi_b827ebb532a7_5a494f4e_managed_psk
...
connmanctl> quit
```

# Optional: Configure Bluetooth
```shell
$ ssh root@<ip-address>
$ lsmod | grep bluetooth
bluetooth             491520  26 hci_uart,btbcm,bnep
ecdh_generic           16384  1 bluetooth
rfkill                 36864  5 bluetooth,cfg80211
$ btuart
...
$ connmanctl
connmanctl> enable bluetooth
Enabled bluetooth
connmanctl> quit
$ bluetoothctl
Agent registered
[bluetooth]# default-agent
Default agent request successful
[bluetooth]# power on
Changing power on succeeded
[bluetooth]# show
...
[bluetooth]# scan on
[bluetooth]# scan off
...
[CHG] Controller DC:A6:32:0A:8E:AF Discovering: no
[bluetooth]# pair DC:08:0F:03:52:CD
[bluetooth]# connect DC:08:0F:03:52:CD
```
