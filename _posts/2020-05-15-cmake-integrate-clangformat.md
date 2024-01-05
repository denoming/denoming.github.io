---
title: "CMake: Integrate clang-format tool"
date: 2020-05-15
categories: [Development,CMake]
tags: [cmake,clang-format]
---

# Prerequisites

Install `ClangFormat` in the system:
```bash
$ sudo apt install -y clang-format-9
```
We choose the last version of `clang-format` to install.

# Integrate

Download and place [FindClangFormat.cmake](https://github.com/karz0n/warehouse/blob/8e00ddc7893ba0f619f94824d4f60a388ad39b8b/cmake/modules/FindClangFormat.cmake) and [CodeFormat.cmake](https://github.com/karz0n/warehouse/blob/12e0b441b09a37db6114e044abfac764e6344cee/cmake/modules/CodeFormat.cmake) module files at the project.

**Important:** Don't forget to include dir there you place these module files to `CMAKE_MODULE_PATH`. As example:
```cmake
list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake/modules")
```

Then include it:
```cmake
include(CodeFormat)
```

After that additional targets will be available and you can use them:
```bash
$ cmake -S . -Bbuild # To configure build directory
$ cmake --build build --target formatcheck # To check code format
```
or
```bash
$ cmake -S . -Bbuild # To configure build directory
$ cmake --build build --target codeformat # To format code inplace
```

Don't forget about clang format file. As example you can use this [one](https://github.com/karz0n/warehouse/blob/12e0b441b09a37db6114e044abfac764e6344cee/cfg/.clang-format). Place it at the root of your project.

Enjoy.



