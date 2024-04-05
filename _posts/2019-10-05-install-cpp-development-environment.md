---
title: "Install C++ development environment on Ubuntu"
date: 2019-10-05
categories: [Development,Configure]
tags: [gcc,cmake,clang,ubuntu]
---

# Install build essential

```bash
$ sudo apt update
$ sudo apt install -y build-essential
```

# Install GCC

## Install GCC on Ubuntu 22

Install the desired GCC/G++ (e.g. the latest one):

```bash
$ sudo apt install gcc-12 g++-12
```

Update alternatives for convenient switching:

```bash
$ sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 11 --slave /usr/bin/g++ g++ /usr/bin/g++-11 --slave /usr/bin/gcov gcov /usr/bin/gcov-11
$ sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-12 12 --slave /usr/bin/g++ g++ /usr/bin/g++-12 --slave /usr/bin/gcov gcov /usr/bin/gcov-12
```

Configure alternatives (by default the highest priority alternative used):

```bash
$ sudo update-alternatives --config gcc
```

Output:

```
There are 2 choices for the alternative gcc (providing /usr/bin/gcc).

  Selection    Path             Priority   Status
------------------------------------------------------------
  0            /usr/bin/gcc-12   12        auto mode
* 1            /usr/bin/gcc-11   11        manual mode
  2            /usr/bin/gcc-12   12        manual mode
```

## Install GCC on Debian 11.5

Add additional packages source list:
```shell
$ sudo tee /etc/apt/sources.list.d/testing.list > /dev/null <<EOF
deb http://deb.debian.org/debian/ testing main non-free contrib
deb-src http://deb.debian.org/debian/ testing main non-free contrib
EOF
```

Configure packages preferences:
```shell
$ sudo tee /etc/apt/preferences.d/stable > /dev/null <<EOF
Package: *
Pin: release a=stable
Pin-Priority: 700
EOF
$ sudo tee /etc/apt/preferences.d/testing > /dev/null <<EOF
Package: *
Pin: release a=testing
Pin-Priority: 1
EOF
```

Install GCC:
```shell
$ sudo apt -y install gcc-11
```

# Install CLang

```bash
$ sudo apt update
$ sudo apt install -y clang-14 lldb-14 lld-14 clangd-14
```

Using clang by cmake:

```
$ cmake -S . -B build \
-DCMAKE_C_COMPILER=/usr/bin/clang-14 \
-DCMAKE_CXX_COMPILER=/usr/bin/clang++-14
```

# Install CMake

```bash
$ sudo apt install -y libssl-dev
$ wget https://github.com/Kitware/CMake/releases/download/v3.24.3/cmake-3.24.3.tar.gz
$ tar xf cmake-3.24.3.tar.gz
$ cd cmake-3.24.3
$ ./bootstrap
$ make -j$(nproc)
$ sudo make install
```
