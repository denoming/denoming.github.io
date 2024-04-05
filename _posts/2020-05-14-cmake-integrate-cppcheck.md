---
title: "CMake: Integrate cppcheck tool"
date: 2020-05-14
categories: [Development,CMake]
tags: [cmake,cppcheck]
---

# Prerequisites

Install `cppcheck` in the system:
```bash
$ sudo apt install cppcheck
```

# Integrate

Download and place [FindCppCheck.cmake](https://github.com/karz0n/warehouse/blob/95bf499bf8dabc381caa11e6093e35cc32ce0ede/cmake/modules/FindCppCheck.cmake) and [CodeAnalysis.cmake](https://github.com/karz0n/warehouse/blob/12e0b441b09a37db6114e044abfac764e6344cee/cmake/modules/CodeAnalysis.cmake) module files at the project.

**Important:** Don't forget to include dir there you place these module files to `CMAKE_MODULE_PATH`. As example:
```cmake
list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake/modules")
```

Then include it:
```cmake
include(CodeAnalysis)
```

After that additional target will be available and you can invoke it from **build** directory:
```bash
$ cmake -S . -Bbuild # To configure build directory
$ cmake --build build --target codeanalysis # To make code analysis
```

Enjoy.



