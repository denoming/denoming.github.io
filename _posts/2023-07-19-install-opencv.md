---
title: "Install OpenCV"
date: 2023-06-07
categories: [Engineering,Linux]
tags: [opencv]
---


# Install build environment

```shell
$ sudo apt install build-essential cmake
```

# Install support packages

Enable V4L Support
```shell
$ sudo apt install -y v4l-utils libv4l-dev
```

Enable Media Support:
```shell
# apt install \
libjpeg-dev libpng-dev libtiff5-dev libxine2-dev libtbb-dev libopencore-amrnb-dev libopencore-amrwb-dev \
libmp3lame-dev libtheora-dev libxvidcore-dev libx264-dev x264
```

# Enable Python Support
```shell
sudo apt install python2 python3
```

# Enable GTK Support
```shell
apt install libgtk-3-dev
```

# Enable FFMPEG Support
```shell
apt install libavcodec-dev libavformat-dev libavutil-dev libswscale-dev
```

# Enable GStreamer Support
```shell
apt install gstreamer1.0* libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev
apt install ubuntu-restricted-extras
```

$ Building

```shell
$ wget -O opencv.zip https://github.com/opencv/opencv/archive/4.x.zip
$ unzip opencv.zip
$ cd opencv-4.x
$ cmake -B build -DCMAKE_INSTALL_PREFIX=$HOME/.local \
-DWITH_TBB=ON -DBUILD_TESTS=OFF -DBUILD_PERF_TESTS=OFF
$ cmake -B build -LH # To list available options
$ cmake --build build --parallel
$ cmake --install build --prefix $HOME/.local
```
