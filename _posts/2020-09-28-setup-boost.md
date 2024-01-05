---
title: "Set-Up Boost library"
date: 2020-09-06
categories: [Development,Bootstrap]
tags: [boost,cmake]
---

1. Download and unpack:
```bash
$ wget https://boostorg.jfrog.io/artifactory/main/release/1.81.0/source/boost_1_81_0.tar.gz
$ tar -xf boost_1_81_0.tar.gz && rm boost_1_81_0.tar.gz
$ cd boost_1_81_0
```
2. Build and install.

Install to the specific location:
```bash
$ ./bootstrap.sh --prefix=${HOME}/.local
$ ./b2 install
```
or to the default location:
```bash
$ ./bootstrap.sh
$ sudo ./b2 install
```
3. Use by [CMake](https://cmake.org/cmake/help/latest/module/FindBoost.html)
