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

## Install in Ubuntu

Install NVIDIA drivers (optional)
```shell
$ sudo ubuntu-drivers list
nvidia-driver-470
$ sudo ubuntu-drivers install nvidia:570
```

Check if NVIDIA driver installed:
```shell
$ nvidia-smi
NVIDIA-SMI 570.133.07
```

Install CUDA toolkit:
```shell
sudo apt install cuda-toolkit
```

## Install in Debian

### Offline installation 

Download run file and run following instructions:
```shell
$ export CUDA_URL=https://developer.download.nvidia.com/compute/cuda/12.4.1/local_installers/cuda_12.4.1_550.54.15_linux.run
$ wget $CUDA_URL
$ chmod +x cuda_12.4.1_550.54.15_linux.run
$ sudo bash cuda_12.4.1_550.54.15_linux.run
```

Download cuDNN tarball and install manually:
```
$ CUDNN_URL=https://developer.download.nvidia.com/compute/cudnn/redist/cudnn/linux-x86_64/cudnn-linux-x86_64-9.13.0.50_cuda12-archive.tar.xz
$ sudo mkdir /usr/local/cudnn-9.13
$ wget -qO- $CUDNN_URL | sudo tar xz -C /usr/local/cudnn-9.13 --strip-components=1 
$ sudo update-alternatives --install /usr/local/cudnn cudnn /usr/local/cudnn-9.13 10
$ sudo tee /etc/ld.so.conf.d/cudnn-9-13.conf > /dev/null <<EOF
/usr/local/cudnn
$ sudo ldconfig
```
## Install samples

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

# Docker
Link: [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/index.html)

Setup NVIDIA Container Toolkit repository:
```shell
$ curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
$ curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

Install container toolkit:
```shell
$ sudo apt update
$ export NVIDIA_CONTAINER_TOOLKIT_VERSION=1.19.0-1
$ sudo apt install -y \
      nvidia-container-toolkit \
      nvidia-container-toolkit-base \
      libnvidia-container-tools \
      libnvidia-container1
```

Configure `Docker`:
```shell
$ sudo nvidia-ctk runtime configure --runtime=docker
$ sudo systemctl restart docker
```

Check:
```shell
$ docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi
```

