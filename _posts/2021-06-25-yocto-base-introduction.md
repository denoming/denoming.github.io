---
title: "Yocto base introduction"
date: 2021-06-25
categories: [Engineering,Yocto]
tags: [yocto,bitbake]
---

# Introduction

![bitbake workflow]({{site.utl}}/assets/img/posts/BitBakeWorkflow.jpg)

Source materials in the form of BitBake recipes feed into the system as inputs. The build system uses the metadata to fetch, configure, patch and compile the source code into binary package. The binary packages are assembled inside a staging area before the Linux image or SDK are generated.

Workflow:
1. Define layers for policy, machine and software metadata;
2. Fetch source code of a software project;
3. Extract source code, apply patches, configure and compile the software;
4. Install the build artifacts into a staging area for packaging;
5. Bundle the installed artifacts into a package feed for the root system;
6. Run checks on a binary package feed;
7. Generate the Linux image or an SDK.

## Metadata

Metadata contains not only recipes but also BSPs, policies, patches and other form of configuration files.
In metadata you first specify machine architecture (`MAHCINE` variable). This leads to include particular machine specific config files, named `tunes` from `meta/conf/machine/include` directory.
Metadata need source code to act. Therefore, the first `do_fetch` task is acting to get source files in a different wats.
Policies in metadata are responsible for which features (e.g. systemd init manager), C library implementation (glibc or musl) and package manager are required in a Linux distribution. Each disto layer (e.g. poky) has it's own `con/distro` subdirectory. The `.conf` files inside the directory define the top-level policies fir a distribution.

Metadata layers:
* __distro__: OS features (init manager, C library and window manager)
* __machine__: CPU architecture, kernel, drivers and bootloader (all hardware specific stuff)
* __recipe__: binaries and/or scripts
* __image__: development, manufactoring or production

Above layers is offering a guidance whe desigining own project to avoid assembling everuthing in one single layer.

# Building

## Building recipe

```shell
$ bitbake <recipe-base-name>
```

The following taks are running by `bitbake` task scheduler to build particular recipe:
* __do_fetch__ - fetches the source code
* __do_patch__ - locates patch files and applies them to the source code
* __do_configure__ - configures the source
* __do_compile__ - compiles the source code in the compilation dir
* __do_install__ - copies files from the compilation dir to a staging area
* __do_package__ + __do_package_data__ - analyzes the content of the staging area and splits into subsets
* __do_package_write_(rpm or deb or ipk)__ - creates the actial packages (RPM or DEB or IPK) and places them in the `Package Feed` area

After __do_compile__ is done, the next __do_install__ task copies resulting files to a staging are where they are readied for packaging.
The tasks __do_package__ and __do_package_data__ are responsible to process the build artifacts in the staging area and divide these files into packages.
The task __do_package_write__ is responsible to create packages and place outcome into package feeds area `tmp/deploy/<package-type>/<package-arch>`.

Main recipe variables:
* `S` - contains the unpacked source files for a fiven recipe
* `D` - destination directory (the directory where the files are installed, before creating the image)
* `T` - temorary dir for log and run files
* `WORKDIR` - location where the OE build system builds a recipe
* `PN` - the name of the recpe used to build the package
* `PV` - the version of the recipe used to build the package
* `PR` - the revision of the recipe to build the package

Work area particular recipe locates at "${TMPDIR}/work/${MULTIMACH_TARGET_SYS}/${PN}/${EXTENDPE}${PV}-${PR}" dir. This dir contains following folders:
* `image` - outcome of `do_install` task 
* `package` and `packages-split` - outcome of `do_package` task as a result of splitting files

The following variables controls splitting installed artifacts to various packages:
* `PACKAGES` - lists all of the packages to be produced
* `FILES` - specifies which files to include in each package by using an override syntax (e.g. `FILES:${PN} += <path/to/folder1> <path/to/folder2>`)

he following variables specifies locations to search include files:
* `FILESPATH` - default value points to `${BP}`, `${BPN}` and `files` subfolders (do not edit manually)
* `FILESOVERRIDES` - extends search path list depending on `${MACHINE}` and `${DISTRO}` values (see `poky/meta/conf/bitbake.conf` file)
* `FILESEXTRAPATHS` - extends search path list (e.g. `FILESEXTRAPATHS:prepend := ${THISDIR}/<some-folder>:`)

The configuration by default is present at `poky/meta/conf/bitbake.conf` file. This files defines variables to ordinary locations, default `FILES` variables for each package and other stuff. 

To configure build-time or runtime dependencies use following variables:
* `DEPENDS` - specifies build-time dependencies, via a list of bitbake recipes to build prior to build the recipe
* `RDEPENDS:${PN}` - specifies run-time dependencies, via a list of packages to install prior to installing the current package  

To provide build-time or runtime aliases use following variables:
* `PROVIDES` - specifies a list of build-time aliases (e.g. `PROVIDES:append = fullkeyboard` for `keyboard` recipe)
* `RPROVIDES:${PN}` - specifies a list of runtime aliases (e.g. `RPROVIDES:${PN} = jpeg` for `libjpeg-turbo` recipe)

To log any information in recipe there are following methods:
* within python script: `bb.{fatal|error|warn|note|plain|debug}`
* within shell script: `bbfatal | bberror | bbwarn | bbnote | bbplain | bbdebug`

To specify recipe compatibility use following statements:
* `COMPATIBLE_MACHINE` - specify a list of supported machines (e.g. `COMPATIBLE_MACHINE = "qemux86|qemux86-64"`)
* `COMPATIBLE_HOST` - specify a list of supported build machines (e.g. `COMPATIBLE_HOST = "qemuarm"`)

In the following example we allow x86 and x86-64 but disallow all others:
```
COMPATIBLE_MACHINE = "(-)"
COMPATIBLE_MACHINE_x86 = "(.*)"
COMPATIBLE_MACHINE_x86-64 = "(.*)"
```
In the following example we turn off specific machines:
```
COMPATIBLE_MACHINE_armv4 = "(!.*armv4).*"
COMPATIBLE_MACHINE_armv5 = "(!.*armv5).*"
```

## Building image

```shell
$ bitbake core-image-minimal
```

Generating an image is multi-stage process that relies on several variable to perform a series of tasks. The task __do_rootfs__ task creates the root filesystem for an image.

Following variables determine what packages get installed onto the image:
* **IMAGE_INSTALL** - the list of packages
* **PACKAGE_EXCLUDE** - the list of packages to omit
* **IMAGE_FEATURES** - the list of additional packages to install (composed as features)
* **PACKAGE_CLASSES** - the type of packages to include
* **IMAGE_LINGUAS** - the list of languages to support

# Running

To run image by QEMU the following packages are required:
* QEMU packages
* `chrpath` package
* `cpio` package
* `diffstat` package
* `rpcgen` (or `rpcsvc-proto` for ArchLinux ditro) package

```shell
$ source sources/poky/oe-init-build-env build
$ sudo env "PATH=$PATH" runqemu-gen-tapdevs `id -u` `id -g` 4 tmp/sysroots-components/x86_64/qemu-helper-native/usr/bin
$ runqemu tmp/deploy/images/qemuarm nographic
or
$ runqemu qemuarm core-image-minimal kvm
or
$ runqemu wic kvm nographic qemuparams="-cpu host -smp 8 -monitor telnet:127.0.0.1:1234,server,nowait -serial telnet:localhost:4321,server,nowait" <path-to-qemuboot.conf-file>
...
```
# Structure

## Source directory

The source directory (`$HOME/sources`) contains at least `poky` directory whic in turn contains core layers:
* __poky/meta__ - the is OpenEmbedded core and contains some changes for poky;
* __poky/meta-poky__ - this is the metadata specific to the poky distribution;
* __poky/meta-yocto-bsp__ - thie contains the board support packages for the machines.

The files of these layers can be found at `sources/poky` directory. The list of layers in which bitbake searches for recipes is stored in `<build-directory>/conf/bblayers.conf`.

Each layers might contain following files:
* __Recipe__ (files ending in .bb) - contains information about building a unit of software (e.g. how to get a copy of source code, the dependencies, how to build and install)
* __Append__ (files ending in .bbappend) - allows some details of a recipe to be overridden or extended
* __Include__ (files ending in .inc) - contains information the is common to several recipes and cen be included by `include` or `required` keyword.
* __Classes__ (files ending in .bbclass) - contains common build information (e.g. hot to build an autotools or cmake project). The classes are inherited by `inherit` keyword.
* __Configuration__ (files ending in .conf) - defines various configuration variables that govern the build process.

The recipe is a collection of tasks written in a combination of Python and shell script. The tasks have names such as **do_fetch**, **do_unpack**, **do_patch**, **do_configure**, **do_compile** and **do_install**. To list available tasks in particular recipe: `bitbake -c listtasks <recipy-name>`.

## Build directory

- __build__
 - __cache__
 - __conf__
     - bblayers.conf - lists the metadata layers to be considered for this project
     - local.conf - contains the project-specific configuration variables
     - templateconf.cfg - contains the directory that includes the template configuration files used to create the project
     - site.conf - contains project specific common configuration variables (not present by default);
 - __downloads__ - contains all the source downloaded for the build
 - __sstate-cache__ - dir with build cache 
 - __tmp__ - build artifacts
	 - work - contains the build dir and the staging area for the root filesystem
	 - deploy - contains the final binaries to be deployed on the target (deploy/images/`<machine-name>` - contains the bootloader, the kernel and the root filesystem images ready to be run on the target)
	 - deploy/rpm - contains the RPM packages the make up the images
	 - deploy/licenses - contains the license files that are extraced from each package		 

## Meta layer

```
meta-example
├── classes
│   ├── class-a.bbclass
│   ├── ...
│   └── class-z.bbclass
├── conf
│   └── layer.conf
├── COPYING.MIT
├── README
├── recipes-a
│   ├── package-a
│   │   └── package-a_0.1.bb
│   ├── ...
│   └── package-z
│       └── package-z_0.1.bb
├── recipes-b
│   └── ...
└── recipes-c
    └── ...
```

# Images

By default there are following standard images:
 * __core-image-minimal__ - provides small console-based system that is isefull for tests and the basis for custom images. The image just capable of allowing a device to boot.
 * __core-image-full-cmdline__ - provides console-only system with more full-featured Linux system functionality installed with full support for the target hardware.
 * __core-image-lsb__ - provides console-only system that is based on Linux Standard Base (LSB).
 * __core-image-x11__ - provides basic X11 Windows system with a graphical terminal.
 * __core-image-sato__ - provides X11 Window system with a SATO theme and a mobile desktop environment.
 * __core-image-weston__ - provides a very basic Wayland image with a terminal.

Additional suffixes can be placed:
 * dev - image suitable for development, as it contains headers and libraries
 * sdk - image includes a complete SDK for development
 * initramfs - image can be used for a RAM-based root filesystem

More information is available at (reference manual)[https://www.yoctoproject.org/docs/2.4/ref-manual/ref-manual.html#ref-images].

# Optimizations

## BitBake

* persist BitBake server during some amount of time by `BB_SERVER_TIMEOUT`

### Disk access optimization

* place the build directories (`build`, `downloads` and `sstate-cache` directories) on their own disk partition or a fast external drive
* use `ext4` file-system without using journalism
* mount partition with specific options - `noatime,barrier=0,commit=6000`

### Disk space optimization

In order to reduce occupied disk space and clean temporary work dir `build/tmp/work` you need to perform following steps:
* Add following lines to `conf/local.conf`
```
INHERIT += "rm_work"
```
* Exclude any recipe (optionally):
```
RM_WORK_EXCLUDE += "busybox glibc"
```

## Scenarios

### Allow empty package

Add following row in the particular recipe:
```
ALLOW_EMPTY:${PN} = "1"
or
ALLOW_EMPTY:${PN}-dev = "1"
or
ALLOW_EMPTY:${PN}-staticdev = "1"
```

Above there rows demostrates how to allow emptiness for `${PN}` or `${PN}-dev` or `${PN}-staticdev` packages that is generated by particular recipe.

### Create custom layer

```shell
$ source sources/poky/oe-init-build-env build-qemuarm
$ bitbake-layers create-layer ../sources/meta-custom
$ bitbake-layers add-layer ../sources/meta-custom
$ bitbake-layers show-layers
layer                 path                                      priority
==========================================================================
meta                  /home/denys/data/yocto/minimal/sources/poky/meta  5
meta-poky             /home/denys/data/yocto/minimal/sources/poky/meta-poky  5
meta-yocto-bsp        /home/denys/data/yocto/minimal/sources/poky/meta-yocto-bsp  5
meta-custom           /home/denys/data/yocto/minimal/sources/meta-custom  6
```

### Check layer for compatibility

```shell
$ yocto-check-layer <path-to-layer-dir>
```

### Create custom image recipe

We assume the custom layer has been created and added to the `bblayers.conf` file.
```shell
$ source sources/poky/oe-init-build-env build-qemuarm
$ cd $PWD/../sources
$ mkdir -p meta-custom/recipes-local/images
$ tee meta-custom/images/custom-image.bb > /dev/null <<EOF
require recipes-core/images/core-image-minimal.bb
IMAGE_INSTALL += "helloworld"
EOF
```

### Pass environment variables

Running bitbake and set the list of variables:
```
export PROD=1
export BB_ENV_EXTRAWHITE="$BB_ENV_EXTRAWHITE PROD"
bitbake ...
```

Using variables in the recipe:
```
DISTRO_FEATURES =. ${@oe.utils.conditional('PROD', '1', '', 'devel ', d)}
```

### Enable package management

First you need to enable package managemenet in general as image feature:
```shell
$ vim conf/local.conf
...
IMAGE_FEATURES:append = "package-management"
$ bitbake <image-name>
```
After this you need to flash obtained image and check existent of `opkg` tool on your RaspberryPi.

Next, will be helpful to organize package installing from local server. As example, we will install `curl` package from our server. To do this we need to setup and run local package server:
```
$ bitbake curl
$ bitbake package-index
$ ls -1 tmp/deploy/ipk
all
cortexa53
raspberrypi3_64
$ cd tmp/deploy/ipk
$ sudo python3 -m http.server --bind <local-ip-address> 80
Serving HTTP on 192.168.1.20 port 80 (http://local-ip-address/) ...
```
After that you will have local package-manager server.

Next, we need to configure `opkg` on our RaspberryPi device:
```shell
$ ssh root@<rpi3-ip-address>
$ vi /etc/opkg/opkg.conf
src/gz all http://<host-ip-address>/all
src/gz cortexa53 http://<host-ip-address>/cortexa53
src/gz raspberrypi3_64 http://<host-ip-address>/raspberrypi3_64
$ opkg update
Downloading http://<host-ip-address>/all/Packages.gz.
Updated source 'all'.
Downloading http://<host-ip-address>/cortexa53/Packages.gz.
Updated source 'cortexa53'.
Downloading http://<host-ip-address>/raspberrypi3_64/Packages.gz.
Updated source 'raspberrypi3_64'.
```

Now we are ready to install `curl` package from our server:
```shell
$ curl
-sh: curl: command not found
$ opkg install curl
Installing curl (7.84.0) on root
Downloading http://<ip-address-of-server>/cortexa53/curl_7.84.0-r0_cortexa53.ipk.
Configuring curl.
$ curl --help
...
```

Futher, you are able to update any already installed package by `opkg` tool:
```shell
$ opkg list-upgradable
...
$ opkg upgrade
```

The same way you are able to setup local package-manager server to use with QEMU instance or docker container. The only difference is the `opkg` config file:
```text
src/gz all http://<host-ip-address>:80/all
src/gz core2-64 http://<host-ip-address>:80/core2-64
src/gz qemux86_64 http://<host-ip-address>:80/qemux86_64
```

### Enable debug mode

To enable debug mode add next lines to the `conf/local.conf` file:

```text
EXTRA_IMAGE_FEATURES += "\
      dbg-pkgs \       # adds -dbg packages for all installed packages and symbol information for debugging and profiling.
      tools-debug \    # adds debugging tools like gdb and strace.
      tools-profile \  # add profiling tools (oprofile, exmap, lttng valgrind (x86 only))
      tools-testapps \ # add useful testing tools (ts_print, aplay, arecord etc.)
      debug-tweaks \   # make image for suitable of development, like setting an empty root password
      tools-sdk \      # OPTIONAL: adds development tools (gcc, make, pkgconfig, etc)  
      dev-pkgs"        # OPTIONAL: adds -dev packages for all installed packages

# Specifies to build packages with debugging information
DEBUG_BUILD = "1"

# Do not remove debug symbols
INHIBIT_PACKAGE_STRIP = "1"

# OPTIONAL: Do not split debug symbols in a separate file
INHIBIT_PACKAGE_DEBUG_SPLIT= "1"
```

### Clear building artifacts

To clear source code after building add next line to the `conf/local.conf` file:

```text
INHERIT += "rm_work"
```

This class bring deletion of temporary workspace which can ease hard drive demands during builds.

To keep source code of specific package you can add it exclude list:

```text
RM_WORK_EXCLUDE += "busybox glibc"
```

### Specifing preferences

To specify preferred provider use following statement at `conf/local.conf`:
```
PREFERRED_PROVIDER_<provide-name> = "<recipe>"
```

To specify preffered version use following statement at `conf/local.conf`:
```
PREFERRED_VERSION_<recipe> = "0.1"
```

### Enable systemd

Add following config to `conf/local.conf` file:

```
# Use systemd for system initialization
DISTRO_FEATURES:append = " systemd"
DISTRO_FEATURES_BACKFILL_CONSIDERED:append = " sysvinit"
VIRTUAL-RUNTIME_init_manager = "systemd"
VIRTUAL-RUNTIME_initscripts = "systemd-compat-units"
VIRTUAL-RUNTIME_login_manager = "shadow-base"
VIRTUAL-RUNTIME_dev_manager = "systemd"
```

### Increase free disk space

Add following config to `conf/local.conf` file:

```
# Specify image filesystem types
IMAGE_FSTYPES = "tar.bz2 ext4 rpi-sdimg"

# Increase free disk space
IMAGE_ROOTFS_EXTRA_SPACE = "1048576"
IMAGE_OVERHEAD_FACTOR = "1.5"
```

### Disable SSH host checking

Edit `$HOME/.ssh/config` file and add following config:
```text
Host 192.168.122.* 
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
  User root
  LogLevel QUIET
```

### Flashing the WIC image using `dd` tool

Input:
* `<image-file.wic.xz>`

```shell
$ unxz image-file.wic.xz
$ wic ls image-file.wic
...
$ lsblk
...
$ sudo dd if=image-file.wic of=/dev/sdb bs=4096 status=progress
$ sync
```

For RaspberryPi `*.rpi-sdimg` files:

```bash
$ cd /yocto/build/tmp/deploy/images/raspberrypi3
$ ls -1 *sdimg
...-raspberrypi3-20200524105934.rootfs.rpi-sdimg
...-raspberrypi3.rpi-sdimg
$ df
/dev/sdd1           40314     17405      22910  44% /media/...
/dev/sdd2          693920    181504     474528  28% /media/...
$ sudo umount /dev/sdd1
$ sudo umount /dev/sdd2
$ sudo dd if=...-raspberrypi3.rpi-sdimg of=/dev/sdd bs=1M status=progress
```

### Flashing the WIC image using `bmaptool`

```bash
$ sudo apt-get install bmap-tools
$ sudo umount /dev/sdN
$ sudo bmaptool copy --nobmap <image>.wic.gz /dev/sdN
```

### Partitioning using fdisk tool

```shell
$ umount /dev/sdb1
$ umount /dev/sdb2
$ sudo fdisk /dev/sdb
% d # Delete 1st partition
> 1
% d # Delete 2nd partition
> 2
% n # Create 1st primary partition
> p > 1 > 2048 > +32M
% n # Create 2nd primary partition 
> p > 2 > (default) > (default)
% a # Set 1st partition as bootable
> 1
% t
> 1 > c W95 FAT32 (LBA)
% t
> 2 > 83 (Linux)
% w # Write changes
$ sudo mkfs.vfat -n "BOOT" /dev/sdb1
$ sudo mkfs.ext4 -n "ROOT" /dev/sdb2
```

### Running devshell for recipe

```shell
$ bitbake -c devshell <recipe>

# Show available enviroment
$ echo ${CC}
$ echo ${CXX}
$ echo ${LDFLAGS}
$ echo ${CFLAGS}

# Run particular script
$ bash temp/run.do_compile
```

### Perform offline build

1. Create a clean downloads dir
2. Generate tarballs of the source git repositories using following statements in the `local.conf` file:
```
DL_DIR = "<absolute-path-to-dir>"
BB_GENERATE_MIRROR_TARBALLS = "1"
```
3. Populate donwloads dir without building:
```
$ bitbake <target> --runonly=fetch
```
4. Optionally remove any git or other SCM subdirs from the downloads dir (e.g. `${DL_DIR}/git2`)
5. Before building specify `BB_NO_NETWORK = "1"` and custom downloads dir at `local.conf` file.

### Using yocto `devtool` tool

#### Building

To start working with `devtool` tool you should create a layer and build image:
```shell
$ cd $HOME/yocto
$ mkdir sources
$ git clone -b dunfell git://git.yoctoproject.org/poky.git sources/poky
$ source sources/poky/oe-init-build-env build-mine

$ bitbake-layers create-layer ${PWD}/../sources/meta-mine
$ bitbake-layers add-layer ${PWD}/../sources/meta-mine
$ bitbake-layers show-layers
layer                 path                                      priority
==========================================================================
meta                  /home/denys/workspace/source/yocto-runner/mine/sources/poky/meta  5
meta-poky             /home/denys/workspace/source/yocto-runner/mine/sources/poky/meta-poky  5
meta-yocto-bsp        /home/denys/workspace/source/yocto-runner/mine/sources/poky/meta-yocto-bsp  5
meta-mine             /home/denys/workspace/source/yocto-runner/mine/sources/meta-mine  6

$ devtool build-image core-image-full-cmdline
INFO: Successfully built core-image-full-cmdline. You can find output files in ...build-mine/tmp/deploy/images/qemux86-64

# Boot image
$ runqemu qemux86-64 nographic kvm
...
qemux86-64 login:
$ ssh root@192.168.7.2
```

#### Creating a new recipe

```shell
$ cd $HOME/yocto
$ source sources/poky/oe-init-build-env build-mine
$ devtool add https://github.com/containers/bubblewrap/releases/download/v0.6.2/bubblewrap-0.6.2.tar.xz
...
INFO: Recipe .../build-mine/workspace/recipes/bubblewrap/bubblewrap_0.6.2.bb has been automatically created; further editing may be required to make it fully functional
$ devtool edit-recipe bubblewrap
...
FILES_${PN} += "/usr/share/*"
$ devtool build bubblewrap
$ devtool deploy-target bubblewrap root@192.168.7.2
...
INFO: Successfully deployed .../build-mine/tmp/work/core2-64-poky-linux/bubblewrap/0.6.2-r0/image
$ ssh root@192.168.7.2
% bwrap --help
...
% exit
$ devtool undeploy-target bubblewrap root@192.168.7.2
...
INFO: Successfully undeployed bubblewrap
$ devtool finish -f bubblewrap ${PWD}/../sources/meta-mine
$ rm -rf workspace/sources/bubblewrap
```

#### Modifying the source built by a recipe

```shell
$ cd $HOME/yocto
$ source sources/poky/oe-init-build-env build-mine
$ bitbake-layers show-layers
...
meta                  /home/denys/workspace/source/yocto-runner/mine/sources/poky/meta  5
meta-poky             /home/denys/workspace/source/yocto-runner/mine/sources/poky/meta-poky  5
meta-yocto-bsp        /home/denys/workspace/source/yocto-runner/mine/sources/poky/meta-yocto-bsp  5
meta-mine             /home/denys/workspace/source/yocto-runner/mine/sources/meta-mine  6
workspace             /home/denys/workspace/source/yocto-runner/mine/build-mine/workspace  99
$ bitbake-layers remove-layer workspace
$ bitbake-layers remove-layer meta-mine
$ git clone -b dunfell https://github.com/openembedded/meta-openembedded.git ${PWD}/../sources/meta-openembedded
$ bitbake-layers add-layer ${PWD}/../sources/meta-openembedded/meta-oe
$ bitbake-layers add-layer ${PWD}/../sources/meta-mine
$ bitbake-layers show-layers
...
meta                  /home/denys/workspace/source/yocto-runner/mine/sources/poky/meta  5
meta-poky             /home/denys/workspace/source/yocto-runner/mine/sources/poky/meta-poky  5
meta-yocto-bsp        /home/denys/workspace/source/yocto-runner/mine/sources/poky/meta-yocto-bsp  5
meta-oe               /home/denys/workspace/source/yocto-runner/mine/sources/meta-openembedded/meta-oe  6
meta-mine             /home/denys/workspace/source/yocto-runner/mine/sources/meta-mine  6
$ tee -a conf/local.conf > /dev/null <<EOF
# Add dependency for jq
IMAGE_INSTALL:append = " onig"
EOF
$ devtool build-image core-image-full-cmdline
$ runqemu qemux86-64 nographic kvm
...
qemux86-64 login:
$ devtool modify jq
...
INFO: Recipe jq now set up to build from .../build-mine/workspace/sources/jq

# Modify jq source code

$ devtool build jq
$ ssh-keygen -f "$HOME/.ssh/known_hosts" -R "192.168.7.2"
$ devtool deploy-target jq root@192.168.7.2
...
INFO: Successfully deployed .../build-mine/tmp/work/core2-64-poky-linux/jq/1.6-r0/image

# Validate modified version of package

$ devtool undeploy-target jq root@192.168.7.2
$ devtool finish jq ${PWD}/../sources/meta-mine
$ rm -rf workspace/sources/jq
```

#### Upgrading a recipe to a newer version

```shell
$ cd $HOME/yocto
$ source sources/poky/oe-init-build-env build-mine
$ bitbake-layers show-layers
...
meta                  /home/denys/workspace/source/yocto-runner/mine/sources/poky/meta  5
meta-poky             /home/denys/workspace/source/yocto-runner/mine/sources/poky/meta-poky  5
meta-yocto-bsp        /home/denys/workspace/source/yocto-runner/mine/sources/poky/meta-yocto-bsp  5
meta-mine             /home/denys/workspace/source/yocto-runner/mine/sources/meta-mine  6
workspace             /home/denys/workspace/source/yocto-runner/mine/build-mine/workspace  99
$ bitbake-layers remove-layer workspace
$ bitbake-layers remove-layer meta-mine
$ bitbake-layers add-layer ${PWD}/../sources/meta-openembedded/meta-python
$ bitbake-layers add-layer ${PWD}/../sources/meta-mine
$ bitbake-layers show-layers
...
meta                  /home/denys/workspace/source/yocto-runner/mine/sources/poky/meta  5
meta-poky             /home/denys/workspace/source/yocto-runner/mine/sources/poky/meta-poky  5
meta-yocto-bsp        /home/denys/workspace/source/yocto-runner/mine/sources/poky/meta-yocto-bsp  5
meta-oe               /home/denys/workspace/source/yocto-runner/mine/sources/meta-openembedded/meta-oe  6
meta-python           /home/denys/workspace/source/yocto-runner/mine/sources/meta-openembedded/meta-python  7
meta-mine             /home/denys/workspace/source/yocto-runner/mine/sources/meta-mine  6
$ bitbake -s | grep ^python3
...
python3-flask                                       :1.1.1-r0
...
$ tee -a conf/local.conf > /dev/null <<EOF
# Add dependency
IMAGE_INSTALL:append = " python3 python3-flask"
EOF
$ devtool build-image core-image-full-cmdline
$ runqemu qemux86-64 nographic kvm
...
qemux86-64 login:
$ devtool upgrade python3-flask --version 1.1.2
...
INFO: New recipe is .../build-mine/workspace/recipes/python3-flask/python3-flask_1.1.2.bb
$ devtool edit-recipe python3-flask
inherit pypi setuptools3
require python-flask.inc
$ devtool build python3-flask
$ ssh-keygen -f "$HOME/.ssh/known_hosts" -R "192.168.7.2"
$ devtool deploy-target python3-flask root@192.168.7.2
...
INFO: Successfully deployed .../build-mine/tmp/work/core2-64-poky-linux/python3-flask/1.1.2-r0/image
$ ssh root@192.168.7.2
root@qemuarm64:~# python3
>>> import flask
>>> flask.__version__
'1.1.2'
>>> quit()
root@qemuarm64:~# exit
$ devtool finish -f python3-flask ${PWD}/../sources/meta-mine
$ rm -rf workspace/sources/python3-flask
```

## Useful Links

* [All sort of documentation](https://docs.yoctoproject.org/)
* [Variables Glossary](https://docs.yoctoproject.org/3.3.1/ref-manual/variables.html)
* [Index of available layers and recipes](https://layers.openembedded.org/)