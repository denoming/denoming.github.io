---
title: "Embedded Linux Programming: Toolchain"
date: 2022-06-19
categories: [Engineering,Embedded]
tags: [embedded,toolchain]
---

# Introduction

![build pipeline]({{site.utl}}/assets/img/posts/EmbeddedProgramming1.svg)

A standard GNU toolchain consists of three main components:
* binutils (assembler + linked);
* GNU Compiler Collection (GCC);
* C library API based on the POSIX specification.

There are following types of toolchains:
* native (same type of system to run and compile target system);
* cross (different type of system to run and compile).

Each toolchain satisfies several CPU architectures characteristics:
* CPU architecture (e.g. ARM);
* big- or little-endian;
* floating point support (hardware supported or configured to use floating-point library);
* application binary interface (ABI) - the calling convention used for passing parameters between function call.

Depending on CPU architectures characteristics each toolchain has particular name. It consists of a tuple of three or four components separated by dashes.

## Naming format

CPU:
* eb (big-endian) => **armeb**
* el (little-endian) => **mipsel**

Vendors:
* buildroot
* poky
* unknown

Operating System (user space):
* gnu (e.g. gnueabi / gnueabihf)
* musl (e.g. musleabi / musleabihf)

Binary interface:
* eabi - uses general-purpose (integer) registers only
* eabihf - uses floating point registers for passing floating parameters.

Example:
```shell
$ gcc -dumpmachine
x86_64-linux-gnu
```

This tuple indicates:
* x86_64 - CPU
* linux - kernel
* gnu - user space

When a native compiler is installed on a machine, it's normal to create links to each of the tools in the toolchain with no prefixes, like: `gcc`.

Here is an example using a cross compiler:
```shell
$ mipsel-unknown-linux-gnu-gcc -dumpmachine
mipsel-unknown-linux-gnu
```
This tuple indicates:
* MIPS - CPU
* uknown - vendor
* linux - kernel
* gnu - user space

Additional information can be obtained by following commands:
* `gcc --version` - get verbose information about the version of GCC
* `gcc -v` - get the verbose information about configuration (like what is the sysroot, CPU, etc)
* `gcc --target-help` - get the list of target specific options
* `gcc -print-sysroot` - get that path to embended system root (can be empty)

# Prepare

Create artifacts folder:

```shell
$ export TOP_DIR=$HOME/toolchains/crosstool-ng
$ mkdir -p $TOP_DIR
```

Build docker image:

```shell
$ git clone https://github.com/karz0n/warehouse.git
$ cd warehouse/docker/crosstool-ng
$ docker build \
--build-arg USERNAME=$USER \
--build-arg USER_UID="$(id -u)" \
--build-arg USER_GID="$(id -g)" \
--tag crosstool-ng .
```

Run container:

```shell
$ docker run -it \
--hostname local \
--rm \
--volume $TOP_DIR:$HOME/toolchains/crosstool-ng \
--env CT_PREFIX=$HOME/toolchains/crosstool-ng \
 crosstool-ng /bin/bash
```

Useful files:

* [Dockerfile](https://github.com/karz0n/warehouse/blob/master/docker/crosstool-ng/Dockerfile)
* [environment.arm](https://github.com/karz0n/warehouse/blob/master/docker/crosstool-ng/environment.arm)
* [environment.arm64](https://github.com/karz0n/warehouse/blob/master/docker/crosstool-ng/environment.arm64)

# Build toolchain

Building of cross toolchain should be performed inside the container.

## For ARM architecture (QEMU):

```shell
$ ct-ng distclean
$ ct-ng arm-unknown-linux-gnueabi
$ ct-ng menuconfig
# Go to "Paths and misc options":
# - disable option "Render the toolchain read-only" (find by CT_PREFIX_DIR_RO)
# - set path CT_LOCAL_TARBALLS_DIR="${CT_PREFIX}/src"
$ ct-ng build.8
```

The outcome will locate at `$HOME/toolchains/arm-unknown-linux-gnueabi` folder.

Create cross toolchain environment file:

```shell
$ tee > /dev/null environment.arm <<'EOF'
export PATH=$HOME/toolchains/arm-unknown-linux-gnueabi/bin:$PATH
export CROSS_COMPILE=arm-unknown-linux-gnueabi-
export ARCH=arm
EOF
```

## For ARM64 architecture (Raspberry Pi 3):

```shell
% ct-ng distclean
% ct-ng aarch64-rpi3-linux-gnu
% ct-ng menuconfig
# Go to "Paths and misc options":
# - disable option "Render the toolchain read-only" (find by CT_PREFIX_DIR_RO)
# - set path CT_LOCAL_TARBALLS_DIR="${CT_PREFIX}/src"
% ct-ng build.8
```

The outcome will locate at `$TOP_DIR/aarch64-rpi3-linux-gnu` folder.

Create cross toolchain environment file:

```shell
$ tee > /dev/null environment.arm64 <<'EOF'
export PATH=$HOME/toolchains/aarch64-rpi3-linux-gnu/bin:$PATH
export CROSS_COMPILE=aarch64-rpi3-linux-gnu-
export ARCH=arm64
EOF
```

# Using of toolchain

As example where we are going to cross compile SQLite3:

```shell
$ export SYSROOT=$(arm-unknown-linux-gnueabi-gcc -print-sysroot)
$ source environment.arm
$ wget https://www.sqlite.org/2022/sqlite-autoconf-3380500.tar.gz
$ tar xf sqlite-autoconf-3380500.tar.gz
$ cd sqlite-autoconf-3380500
$ CC=arm-unknown-linux-gnueabi-gcc \
$ ./configure --host=arm-cortex_a5-linux-gnueabihf --prefix=/usr
$ make
$ make DESTDIR=$SYSROOT install
$ PKG_CONFIG_LIBDIR=$SYSROOT/usr/lib/pkgconfig pkg-config sqlite3 --libs --cflags
-lsqlite3
```
