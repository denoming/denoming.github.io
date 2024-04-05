---
title: "Install OpenCV"
date: 2023-06-07
categories: [Engineering,Linux]
tags: [ComputerVision,opencv]
---

# Install prerequisites

## Ubuntu

```shell
# Install build environment
$ sudo apt install build-essential cmake
# Enable V4L Support
$ sudo apt install -y v4l-utils libv4l-dev
# Enable Media Support
$ sudo apt install \
libjpeg-dev libpng-dev libtiff5-dev libxine2-dev libtbb-dev libopencore-amrnb-dev libopencore-amrwb-dev \
libmp3lame-dev libtheora-dev libxvidcore-dev libx264-dev x264
# Enable Python Support
sudo apt install python2 python3
# Enable GTK Support
$ sudo apt install libgtk-3-dev
# Enable FFMPEG Support
$ sudo apt install libavcodec-dev libavformat-dev libavutil-dev libswscale-dev
# Enable GStreamer Support
$ sudo apt install gstreamer1.0* libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev
$ sudo apt install ubuntu-restricted-extras
```

## Arch

### Install CUDA

```shell
$ sudo pacman -S nvidia nvidia-utils nvidia-settings cuda cudnn
$ lspci -v | grep VGA
01:00.0 VGA compatible controller: NVIDIA Corporation GA102 [GeForce RTX 3080 Lite Hash Rate] (rev a1) (prog-if 00 [VGA controller])
```

Determine compute version using following links:
* [Matching CUDA arch and CUDA gencode for various NVIDIA architectures](https://arnon.dk/matching-sm-architectures-arch-and-gencode-for-various-nvidia-cards/)
* [CUDA GPUs](https://developer.nvidia.com/cuda-gpus)

For example:
```
GeForce RTX 3080
...Compute Capability 8.6
...Ampere (CUDA 11.1 and later)
...SM86 or SM_86, compute_86
```

Install Python 2 (needed for OpenCV 4.8.1):
```shell
$ git clone https://aur.archlinux.org/python2.git
$ cd python2
$ gpg --keyserver keyserver.ubuntu.com --recv-key 04C367C218ADD4FF
$ makepkg -si
```

### Install prerequisites

```shell
$ sudo pacman -S --needed base-devel cmake git python python-numpy openmpi python-mpi4py boost
```

# Build

## Download artifacts
```shell
$ wget -O opencv.zip https://github.com/opencv/opencv/archive/refs/tags/4.8.1.zip
$ unzip opencv.zip && rm opencv.zip
$ wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/refs/tags/4.8.1.zip
$ unzip opencv_contrib.zip && rm opencv_contrib.zip
```

## Without CUDA support
```shell
$ cd opencv-4.8.1
$ cmake -Wno-dev -B build-release \
-DCMAKE_BUILD_TYPE=Release \
-DOPENCV_EXTRA_MODULES_PATH=$HOME/source/opencv_contrib-4.8.1/modules \
-DENABLE_FAST_MATH=ON \
-DWITH_OPENMP=ON \
-DBUILD_EXAMPLES=ON \
-DBUILD_DOCS=OFF \
-DBUILD_PERF_TESTS=OFF \
-DBUILD_TESTS=OFF \
-DWITH_TBB=ON \
-DWITH_IPP=ON \
-DWITH_CSTRIPES=ON \
-DWITH_OPENCL=ON \
-DWITH_FFMPEG=ON \
-DWITH_OPENGL=ON
$ cmake --build build-release
$ cmake --install build-release --prefix $HOME/.local
```

## With CUDA support
```
$ cd opencv-4.8.1
$ cmake -Wno-dev -B build-release \
-DCMAKE_BUILD_TYPE=Release \
-DOPENCV_EXTRA_MODULES_PATH=$HOME/source/opencv_contrib-4.8.1/modules \
-DWITH_CUDA=ON \
-DCUDA_ARCH_BIN=8.6 \
-DARCH=sm_86 \
-Dgencode=arch=compute_86,code=sm_86 \
-DENABLE_FAST_MATH=ON \
-DCUDA_FAST_MATH=ON \
-DWITH_CUBLAS=ON \
-DWITH_CUFFT=ON \
-DWITH_NVCUVID=OFF \
-DWITH_OPENMP=ON \
-DBUILD_EXAMPLES=ON \
-DBUILD_DOCS=OFF \
-DBUILD_PERF_TESTS=OFF \
-DBUILD_TESTS=OFF \
-DWITH_TBB=ON \
-DWITH_IPP=ON \
-DWITH_CSTRIPES=ON \
-DWITH_OPENCL=ON \
-DWITH_FFMPEG=ON \
-DWITH_OPENGL=ON
$ cmake --build build-release
$ cmake --install build-release --prefix $HOME/.local
```