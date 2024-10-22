---
title: "Install DLib"
date: 2024-03-23
categories: [Engineering,Linux]
tags: [ComputerVision,dlib]
---

# Install prerequisites

## Arch

```shell
$ sudo pacman -S --needed cblas cuda cudnn lapack blas libjpeg-turbo libpng libx11 libwebp
```

# Build

Download artifacts:
```shell
$ wget http://dlib.net/files/dlib-19.24.tar.bz2
$ tar xjf dlib-19.24.tar.bz2 && rm dlib-19.24.tar.bz2
$ cd dlib-19.24/dlib
```

Build (with CUDA support + AVX instructions):
```shell
$ cmake -Wno-dev \
-Bbuild-release \
-GNinja \
-DUSE_AVX_INSTRUCTIONS=ON \
-DCMAKE_BUILD_TYPE=Release \
-DBUILD_SHARED_LIBS=ON \
-DDLIB_USE_CUDA=ON \
-DCUDA_HOST_COMPILER=/usr/bin/gcc-13
$ cmake --build build-release --parallel
$ cmake --install build-release --prefix $HOME/.local
```

Note: Specifying `CUDA_HOST_COMPILER` is a workaround as DLib 19.24 seems doesn't support GCC13
