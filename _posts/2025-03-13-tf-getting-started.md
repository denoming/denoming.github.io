---
title: TensorFlow - Getting Started
date: 2025-03-13
categories:
  - MachineLearning
tags:
  - ai
  - tensorflow
---
# Setup

```shell
$ pip install --upgrade pip
$ pip install tensorflow
or
$ pip install tensorflow[and-cuda]
```

## Using Conda

Create environment:
```shell
$ conda create -n env-name python=<version> -y
$ conda activate env-name
```

Install CUDA:
```shell
$ conda install cudatoolkit -c anaconda -y
or
$ conda install cudatoolkit=<version> -c anaconda -y
```

Install TensorFlow:
```shell
# To install TensorFlow
$ conda install tensorflow-gpu cudatoolkit=<version> -y
or
$ pip install "tensorflow[and-cuda]"
or
$ pip install "tensorflow-cpu"
(see https://www.tensorflow.org/install/source#tested_build_configurations for tested build configurations)
```

## Using TensorFlow docker

Use TF+Jupyter docker image:
```shell
$ docker pull tensorflow/tensorflow:latest-jupyter
$ docker run -it -p 8888:8888 -v $PWD:/tf/my tensorflow/tensorflow:latest
```

Use TF+Jupyter docker image with GPU support (docker 19.03 or above + nvidia-container-toolkit are required):
```shell
# Check if a GPU is available
$ lspci | grep -i nvidia
$ docker pull tensorflow/tensorflow:latest-gpu-jupyter
$ docker run -it -p 8888:8888 \
-v $PWD:/tf/my \
--runtime=nvidia --gpus all \
tensorflow/tensorflow:latest-gpu-jupyter
```