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

# Libraries

## Common

| Name                                                                                                           | Notes                                                                                                                                                            |
| -------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [TensorFlow Datasets](https://www.tensorflow.org/datasets)                                                     | Collection of datasets ready to use with TensorFlow. All datasets are exposed as [tf.data.Datasets](https://www.tensorflow.org/api_docs/python/tf/data/Dataset). |
| [Build TensorFlow input pipelines](https://www.tensorflow.org/guide/data)                                      | Build a pipeline from input data and a bunch of pre-processing steps.                                                                                            |
| [Working with preprocessing layers](https://www.tensorflow.org/guide/keras/preprocessing_layers)               | Data pre-processing recipes                                                                                                                                      |
| [Better performance with the tf.data API](https://www.tensorflow.org/guide/data_performance#the_training_loop) | Optimize performance of pipelines                                                                                                                                |
## Vision

| Name                                                                                                                                 | Notes                                                                                                            |
| ------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------- |
| [KerasHub](https://keras.io/keras_hub/)                                                                                              |                                                                                                                  |
| [tf.keras.utils](https://www.tensorflow.org/api_docs/python/tf/keras/utils)                                                          | provides a bunch of useful functions for loading, converting and other high-level image pre-processing utilities |
| [tf.image](https://www.tensorflow.org/api_docs/python/tf/image)<br>[tf.data](https://www.tensorflow.org/api_docs/python/tf/data)<br> | writing custom data augmentation pipelines or layers                                                             |
|                                                                                                                                      |                                                                                                                  |
