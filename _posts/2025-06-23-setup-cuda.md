---
title: Setup CUDA
date: 2025-06-23
categories:
  - My
tags:
  - ml
  - cnn
---
# Install

Install NVIDIA drivers
```shell
$ sudo ubuntu-drivers list
nvidia-driver-470
...
$ sudo ubuntu-drivers install nvidia:570
$ nvidia-smi
NVIDIA-SMI 570.133.07
```

Install CUDA toolkit
```shell
sudo apt install cuda-toolkit
```

## Install CUDA examples

Install prerequisites:
```shell
sudo apt install build-essential cmake
sudo apt install openmpi-bin openmpi-doc libopenmpi-dev
sudo apt install freeglut3-dev libx11-dev libxmu-dev libxi-dev libglu1-mesa-dev libfreeimage-dev libglfw3-dev
```

Clone and build examples:
```shell
git clone https://github.com/NVIDIA/cuda-samples.git
cd cuda-samples
cmake -B build
cmake --build build --parallel
```
Run example:
```shell
build/Samples/1_Utilities/deviceQuery/deviceQuery
```
