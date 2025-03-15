---
title: Using YOLO/DarkNet
date: 2025-03-14
categories:
  - My
tags:
  - yolo
  - darknet
  - cnn
---
# Build

Prepare environment:
```shell
$ apt update
$ apt install build-essential git libopencv-dev libssl-dev
$ mkdir /workspace/src
$ cd /workspace/src
```

Build cmake:
```shell
$ wget https://github.com/Kitware/CMake/releases/download/v3.24.3/cmake-3.24.3.tar.gz
$ tar -xf cmake-3.24.3.tar.gz && rm cmake-3.24.3.tar.gz
$ pushd cmake-3.24.3
$ ./bootstrap --prefix=/usr
$ make -j$(nproc)
$ make install
$ popd && rm -fr cmake-3.24.3
```

Build and install darknet:
```shell
$ cd /workspace/src && rm -fr darknet
$ git clone https://github.com/hank-ai/darknet && cd darknet
$ cmake -B build -D CMAKE_BUILD_TYPE=Release
$ cmake --build build --target package --parallel
$ sudo dpkg -i build/darknet-3.0.212-Linux.deb
$ darknet --version
Darknet V3 "Jazz" v3.0-212-gbb1b3bbb
CUDA runtime version 12040 (v12.4), driver version 12040 (v12.4)
cuDNN version 12040 (v9.1.0), use of half-size floats is ENABLED
=> 0: NVIDIA GeForce RTX 3080 [#8.6], 9.8 GiB
OpenCV v4.5.4, Ubuntu 22.04, docker
```

Build and install DarkHelp:
```shell
$ cd /workspace/src && rm -fr DarkHelp
$ sudo apt install build-essential libopencv-dev libtclap-dev libmagic-dev
$ git clone https://github.com/stephanecharette/DarkHelp.git && cd DarkHelp
$ cmake -B build -D CMAKE_BUILD_TYPE=Release
$ cmake --build build --target package --parallel
$ sudo dpkg -i build/darkhelp-1.9.5-1-Linux-x86_64-Linuxmint-22.1.deb
```

Build and install DarkMark:
```shell
$ sudo apt install build-essential libopencv-dev libx11-dev libfreetype6-dev libxrandr-dev libxinerama-dev libxcursor-dev libmagic-dev libpoppler-cpp-dev
$ git clone https://github.com/stephanecharette/DarkMark.git && cd DarkMark
$ cmake -B build -D CMAKE_BUILD_TYPE=Release
$ cmake --build build --target package --parallel
$ sudo dpkg -i build/Darkmark-1.10.18-1-Linux-x86_64-Linuxmint-22.1.deb
```

# Configre

Modify model `.cfg` file following next rules:
* Make sure that batch=64.
* Note the subdivisions. Depending on the network dimensions and the amount of memory available on GPU, the subdivisions may need to be increased. The best value to use is 1 so start with that.
* Note max_batches=.... A good value to use when starting out is 2000 x the number of classes. For this example, we have 4 animals, so 4 * 2000 = 8000. Meaning we'll use max_batches=8000.
* Note steps=.... This should be set to 80% and 90% of max_batches. For this example we'd use steps=6400,7200 since max_batches was set to 8000.
* Note width=... and height=.... These are the network dimensions.
* Search for all instances of the line classes=... and modify it with the number of classes in your .names file. For this example, we'd use classes=4
* Search for all instances of the line filters=... in the [convolutional] section prior to each [yolo] section. The value to use is (number_of_classes + 5) * 3. Meaning for this example, (4 + 5) * 3 = 27. So we'd use filters=27 on the appropriate lines.

Suggestion for preparing dataset:
* The best ratio between training and validation image sets is 70%/30%. Starting from 20k of `max_batches` with two classes the ratio has little impact on either the loss or the mAP.
* being consistent when marking up images. All instances of a particular class should be marked in the same way
* if the object is not "square" or viewed from and angle, then the markup should includes all parts of the object visialbe in the image
* only mark up the parts of the image which are visible in case some parts of the object is obscured
* mark every single object if multiple objects are appeared in the image
* overlapping object is acceptable
* variation of different images in dataset should be prioritized

# Train

Resize images according to model `*.cfg` file:
```shell
# According to model *.cfg file
# (see "width" and "height" options)
% mogrify -strip -resize 416x416! -quality 75 train/*.jpg
% mogrify -strip -resize 416x416! -quality 75 valid/*.jpg
% mogrify -strip -resize 416x416! -quality 75 test/*.jpg
```

Prepare the list of files in dataset:
```shell
$ find $(pwd)/train -name \*.jpg > track-limits.train
$ find $(pwd)/valid -name \*.jpg > track-limits.valid
```

Train a new network:
```shell
$ darknet detector train -map -dont_show track-limits.data track-limits.cfg | tee output.log
```

Check mAP% results:
```shell
$ darknet detector map track-limits.data track-limits.cfg cones_best.weights
```

Process video using pre-trained model:
```shell
$ darknet_04_process_videos track-limits.cfg video01.mp4
```


