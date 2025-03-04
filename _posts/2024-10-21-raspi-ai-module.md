---
title: Install and Use Raspberry Pi 5 AI Extension
date: 2024-10-21
categories:
  - Engineering
  - Linux
tags:
  - raspberry
  - ai
---

# Introduction

## TAPPAS framework

List of available gstreamer elements:
```text
hailodevicestats: hailodevicestats element
hailonet: hailonet element
hailopython:  hailopython: HailoPython Element
hailoaggregator: hailoaggregator - Cascading
hailocounter: hailocounter - postprocessing element
hailocropper: hailocropper
hailoexportfile: hailoexportfile - export element
hailoexportzmq: hailoexportzmq - export element
hailofilter: hailofilter - postprocessing element
hailogallery: Hailo gallery element
hailograytonv12: hailograytonv12 - postprocessing element
hailoimportzmq: hailoimportzmq - import element
hailomuxer: Muxer pipeline merging
hailonv12togray: hailonv12togray - postprocessing element
hailonvalve: HailoNValve element
hailooverlay: hailooverlay - overlay element
hailoroundrobin: Input Round Robin element
hailostreamrouter: Hailo Stream Router
hailotileaggregator: hailotileaggregator
hailotilecropper: hailotilecropper - Tiling
hailotracker: Hailo object tracking element
```

The main elements that is present in each video pipeline:
* `hailonet`
* `hailofilter`
 
The `hailonet` is responsible for loading model into computation device. There is the main property `hef-path=` to define a path to HEF model file.
The `hailofilter` is responsible for post-processing each video stream frames. There are two main properties: `so-path=` and `function_name=`. The `so-path=` is the path to `*.so` library file and `function_name=` is the entry point (exported function) to use in the given library. By default the `filter` entry point is used. Particular entry point depends on the model that is used. As example, for `mobilenet_v1.hef` model the entry point in `libclassification.so` library is `mobilenet_v1`. The header file of each post-processing library at [source code](https://github.com/hailo-ai/tappas/tree/master/core/hailo/libs/postprocesses) contains the list of entry points.

# Installing

Install NVIDIA container toolkit (to access GPU when training models):
```shell
$ sudo pamac install -y nvidia-container-toolkit
$ sudo nvidia-ctk runtime configure --runtime=docker
$ sudo systemctl restart docker
$ sudo docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi
```

Install `hailo-all` package:
```shell
$ sudo apt update
$ sudo apt install -y hailo-all
$ sudo reboot
```

Check module version:
```shell
$ sudo hailortcli fw-control identify
Executing on device: 0000:01:00.0  
Identifying board  
Control Protocol Version: 2  
Firmware Version: 4.19.0 (release,app,extended context switch buffer)  
Logger Version: 0  
Board Name: Hailo-8  
Device Architecture: HAILO8L  
Serial Number: HLDDLBB241601754  
Part Number: HM21LB1C2LAE  
Product Name: HAILO-8L AI ACC M.2 B+M KEY MODULE EXT TMP
```
## Compile from source

Compile HAilo drivers from source:
```shell
$ git clone https://github.com/hailo-ai/hailort-drivers.git
$ cd hailort-drivers
$ git checkout v4.19.0 -b v4.19.0
cd linux/pcie
$ make all
$ sudo make install
  INSTALL /lib/modules/6.6.63-1-MANJARO/kernel/drivers/misc/hailo_pci.ko
  SIGN    /lib/modules/6.6.63-1-MANJARO/kernel/drivers/misc/hailo_pci.ko
$ sudo modprobe hailo_pci
$ lsmod | grep hailo
hailo_pci             151552  0
$ cd -
$ bash download_firmware.sh
$ sudo mkdir -p /lib/firmware/hailo
...
2024-12-08 22:42:47 (1.43 MB/s) - ‘hailo8_fw.4.19.0.bin’ saved [166260/166260]
$ sudo mv hailo8_fw.4.19.0.bin /lib/firmware/hailo/hailo8_fw.bin
$ sudo cp linux/pcie/51-hailo-udev.rules /etc/udev/rules.d
$ sudo udevadm control --reload-rules && sudo udevadm trigger

```

Compile HailoRT from source:
```shell
$ git clone https://github.com/hailo-ai/hailort.git
$ cd hailort
$ git checkout v4.19.0 -b v4.19.0
$ cmake -Bbuild -Wno-dev \
-DCMAKE_BUILD_TYPE=Release \
-DCMAKE_INSTALL_PREFIX=$HOME/temp/hailort
$ cmake --build build --config release --parallel
$ cmake --install build
```

# Using

## RaspberryPi Camera Applications

Validate the camera is operating:
```shell
$ rpicam-hello -t 10s
```

Object detection:
```shell
# Yolo v6
$ rpicam-hello -t 0 --post-process-file /usr/share/rpi-camera-assets/hailo_yolov6_inference.json --lores-width 640 --lores-height 640
# Yolo v8
$ rpicam-hello -t 0 --post-process-file /usr/share/rpi-camera-assets/hailo_yolov8_inference.json --lores-width 640 --lores-height 640
# Yolo X
$ rpicam-hello -t 0 --post-process-file /usr/share/rpi-camera-assets/hailo_yolox_inference.json --lores-width 640 --lores-height 640
# Yolo v5 (Person and Face model)
$ rpicam-hello -t 0 --post-process-file /usr/share/rpi-camera-assets/hailo_yolov5_personface.json --lores-width 640 --lores-height 640
```

Image segmentation:
```shell
$ rpicam-hello -t 0 --post-process-file /usr/share/rpi-camera-assets/hailo_yolov5_segmentation.json --lores-width 640 --lores-height 640 --framerate 20
```

Poss estimation:
```shell
$ rpicam-hello -t 0 --post-process-file /usr/share/rpi-camera-assets/hailo_yolov8_pose.json --lores-width 640 --lores-height 640
```

## Hailo Model Zoo

Clone repository:
```shell
$ git clone https://github.com/hailo-ai/hailo_model_zoo.git
```

### Re-training YOLO model 

Build image (only once):
```shell
$ cd hailo_model_zoo/training/yolov8
$ docker build \
--build-arg timezone=`cat /etc/timezone` \
--build-arg user=$USER \
--build-arg group=$USER \
--build-arg uid=$(id $USER -u) \
--build-arg gid=$(id $USER -g) \
--tag hailo-yolov8:v0 .
```

Run container (from scratch):
```shell
docker run \
-it --gpus all --ipc=host \
--hostname localhost \
--user "$(id $USER -u):$(id $USER -u)" \
--name "hailo-yolov8" \
-v /home/denys/data/cv/data/:/data \
hailo-yolov8:v0
% yolo detect train data=/data/Barcode-Detection/data.yaml model=yolov8s.pt name=retrain_yolov8s epochs=20 batch=8
```

Run container (resume training):
```shell
$ docker ps -a
CONTAINER ID   IMAGE             COMMAND       CREATED          STATUS          PORTS     NAMES
ee6a90ee77fc   hailo-yolov8:v0   "/bin/bash"   56 minutes ago   Up 28 seconds             hailo-yolov8
$ docker container start ee6a90ee77fc
$ docker attach ee6a90ee77fc
% yolo train resume model=runs/detect/retrain_yolov8s/weights/last.pt
```

Export trained model to ONNX format:
```shell
% yolo export model=runs/detect/retrain_yolov8s/weights/best.pt imgsz=640 format=onnx opset=11
% cp runs/detect/retrain_yolov8s/weights/best.onnx /data/barcode-detection.onnx
```

Compile ONNX model into Hailo model format (HEF file):
```shell
$ cd hailo/AI-Software-Suite
$ bash hailo_ai_sw_suite_docker_run.sh --override
$ hailomz compile yolov8s \
--ckpt=/local/shared_with_docker/data/barcode-detection.onnx \
--hw-arch=hailo8l \
--calib-path=/local/shared_with_docker/data/Barcode-Detection/test/images \
--classes=2 \
--performance
...
<Hailo Model Zoo INFO> HEF file written to yolov8s.hef
$ cp /local/workspace/yolov8s.hef /local/shared_with_docker
```
After obtaining model in HEF format it can be used in inference pipeline. 


## AI Software Suite

```shell
$ unzip hailo_ai_sw_suite_2024-10_docker.zip
# Change following at 'hailo_ai_sw_suite_docker_run.sh' file:
readonly NVIDIA_DOCKER_EXIST=$(dpkg -l | grep 'nvidia-docker\|nvidia-container-toolkit')
To
readonly NVIDIA_DOCKER_EXIST=$(pacman -Q | grep 'nvidia-docker\|nvidia-container-toolkit')
```

## Using TAPPAS framework (Hailo-RPI5)

### Preparing

First, we need to clone and build post-processing libraries. To do that there is a helper script that downloads all resources (video files and model files) and build particular post-processing libraries:
```
$ git clone https://github.com/hailo-ai/hailo-rpi5-examples.git
$ cd hailo-rpi5-examples
$ bash install.sh
```

Define a couple of environment variable after installing script is done:
```shell
$ export VIDEO1=$PWD/resources/detection0.mp4
$ export VIDEO2=$PWD/resources/barcode.mp4
$ export RESOURCES=$PWD/resources
```

#### Object Detection

```shell
$ export MODEL=$RESOURCES/yolov8s_h8l.hef
$ export PROCESS_FN=yolov8s
$ export PROCESS_SO=$RESOURCES/libyolo_hailortpp_postprocess.so
```

```shell
$ gst-launch-1.0 \
	filesrc location=$VIDEO1 ! \
    decodebin ! \
    videoscale n-threads=2 ! \
    videoconvert n-threads=3 qos=false ! \
    queue leaky=no max-size-buffers=30 max-size-bytes=0 max-size-time=0 ! \
    hailonet hef-path=$MODEL batch-size=2 ! \
    queue leaky=no max-size-buffers=30 max-size-bytes=0 max-size-time=0 ! \
    hailofilter so-path=$PROCESS_SO function-name=$PROCESS_FN qos=false ! \
    queue leaky=no max-size-buffers=30 max-size-bytes=0 max-size-time=0 ! \
    hailooverlay qos=false ! \
    videoconvert ! \
    queue leaky=no max-size-buffers=30 max-size-bytes=0 max-size-time=0 ! \
    fpsdisplaysink video-sink=xvimagesink sync=true text-overlay=false signal-fps-measurements=true
```

#### Segmentation

```shell
$ export MODEL=$RESOURCES/yolov5n_seg_h8l_mz.hef
$ export PROCESS_FN=yolov5seg
$ export PROCESS_SO=$RESOURCES/libyolov5seg_postprocess.so
```

```
$ gst-launch-1.0 \
    filesrc location=$VIDEO1 ! \
    decodebin ! \
    videoscale n-threads=2 qos=false ! \
    videoconvert n-threads=2 qos=false ! \
    queue leaky=no max-size-buffers=30 max-size-bytes=0 max-size-time=0 ! \
    hailonet hef-path=$MODEL batch-size=2 ! \
    queue leaky=no max-size-buffers=30 max-size-bytes=0 max-size-time=0 ! \
    hailofilter so-path=$PROCESS_SO function_name=$PROCESS_FN config-path=$RESOURCES/yolov5n_seg.json qos=false ! \
    queue leaky=no max-size-buffers=30 max-size-bytes=0 max-size-time=0 ! \
    hailooverlay qos=false ! \
    videoconvert n-threads=2 qos=false ! \
    queue leaky=no max-size-buffers=30 max-size-bytes=0 max-size-time=0 ! \
    fpsdisplaysink video-sink=xvimagesink sync=true text-overlay=false signal-fps-measurements=true
```

#### Pose estimation

```shell
$ export MODEL=$RESOURCES/yolov8s_pose_h8l.hef
$ export PROCESS_FN=filter
$ export PROCESS_SO=$RESOURCES/libyolov8pose_postprocess.so
```

```
$ gst-launch-1.0 \
    filesrc location=$VIDEO1 ! \
    decodebin ! \ls -1 
    videoscale n-threads=2 qos=false ! \
    videoconvert n-threads=2 qos=false ! \
    queue leaky=no max-size-buffers=30 max-size-bytes=0 max-size-time=0 ! \
    hailonet hef-path=$MODEL batch-size=2 ! \
    queue leaky=no max-size-buffers=30 max-size-bytes=0 max-size-time=0 ! \
    hailofilter so-path=$PROCESS_SO function_name=$PROCESS_FN qos=false ! \
    queue leaky=no max-size-buffers=30 max-size-bytes=0 max-size-time=0 ! \
    hailooverlay qos=false ! \
    videoconvert n-threads=2 qos=false ! \
    queue leaky=no max-size-buffers=30 max-size-bytes=0 max-size-time=0 ! \
    fpsdisplaysink video-sink=xvimagesink sync=true text-overlay=false signal-fps-measurements=true
```

#### Bar codes

```shell
$ export MODEL=$RESOURCES/yolov8s-hailo8l-barcode.hef
$ export PROCESS_FN=filter
$ export PROCESS_SO=$RESOURCES/libyolo_hailortpp_postprocess.so
```

```
$ gst-launch-1.0 \
    filesrc location=$VIDEO2 ! \
    decodebin ! \
    videoscale n-threads=2 qos=false ! \
    videoconvert n-threads=2 qos=false ! \
    queue leaky=no max-size-buffers=30 max-size-bytes=0 max-size-time=0 ! \
    hailonet hef-path=$MODEL batch-size=2 ! \
    queue leaky=no max-size-buffers=30 max-size-bytes=0 max-size-time=0 ! \
    hailofilter so-path=$PROCESS_SO function_name=$PROCESS_FN config-path=$RESOURCES/barcode-labels.json qos=false ! \
    queue leaky=no max-size-buffers=30 max-size-bytes=0 max-size-time=0 ! \
    hailooverlay qos=false ! \
    videoconvert n-threads=2 qos=false ! \
    queue leaky=no max-size-buffers=30 max-size-bytes=0 max-size-time=0 ! \
    fpsdisplaysink video-sink=xvimagesink sync=true text-overlay=false signal-fps-measurements=true
```

## Using TAPPAS framework (Misc)

### Preparing

Create main folders:
```shell
$ mkdir -p hailo/{models,}
$ cd hailo
```

Define variable with path to post-processing libraries:
```shell
$ export PP_LIBS=/usr/lib/aarch64-linux-gnu/hailo/tappas/post_processes
```

#### Classification

* Model: [mobilenet_v1.hef](https://hailo-model-zoo.s3.eu-west-2.amazonaws.com/ModelZoo/Compiled/v2.13.0/hailo8l/mobilenet_v1.hef)

```shell
$ export MODEL=$PWD/models/mobilenet_v1.hef
$ export PROCESS_FN=mobilenet_v1
$ export PROCESS_SO=$PP_LIBS/libclassification.so
```

```shell
$ gst-launch-1.0 \
    libcamerasrc ! video/x-raw,format=NV12,width=640,height=480 ! \
    videoscale n-threads=2 qos=false ! \
    videoconvert n-threads=2 qos=false ! \
    queue leaky=no max-size-buffers=30 max-size-bytes=0 max-size-time=0 ! \
    hailonet hef-path=$MODEL batch-size=2 ! \
    queue leaky=no max-size-buffers=30 max-size-bytes=0 max-size-time=0 ! \
    hailofilter so-path=$PROCESS_SO function_name=$PROCESS_FN qos=false  ! \
    queue leaky=no max-size-buffers=30 max-size-bytes=0 max-size-time=0 ! \
    hailooverlay qos=false ! \
    videoconvert n-threads=2 qos=false ! \
    queue leaky=no max-size-buffers=30 max-size-bytes=0 max-size-time=0 ! \
    fpsdisplaysink video-sink=xvimagesink sync=true text-overlay=false signal-fps-measurements=true
```

#### Face recognition pipeline

* Model: [retinaface_mobilenet_v1.hef](https://hailo-model-zoo.s3.eu-west-2.amazonaws.com/ModelZoo/Compiled/v2.13.0/hailo8l/retinaface_mobilenet_v1.hef)

```shell
$ export MODEL=$PWD/models/retinaface_mobilenet_v1.hef
$ export PROCESS_FN=retinaface
$ export PROCESS_SO=$PP_LIBS/libface_detection_post.so
```

```shell
$ gst-launch-1.0 \
    libcamerasrc ! video/x-raw,format=NV12,width=640,height=480 ! \
    videoscale n-threads=2 qos=false ! \
    videoconvert n-threads=2 qos=false ! \
    queue leaky=no max-size-buffers=30 max-size-bytes=0 max-size-time=0 ! \
    hailonet hef-path=$MODEL batch-size=2 ! \
    queue leaky=no max-size-buffers=30 max-size-bytes=0 max-size-time=0 ! \
    hailofilter so-path=$PROCESS_SO function_name=$PROCESS_FN qos=false  ! \
    queue leaky=no max-size-buffers=30 max-size-bytes=0 max-size-time=0 ! \
    hailooverlay qos=false ! \
    videoconvert n-threads=2 qos=false ! \
    queue leaky=no max-size-buffers=30 max-size-bytes=0 max-size-time=0 ! \
    fpsdisplaysink video-sink=xvimagesink sync=false text-overlay=false signal-fps-measurements=true
```

# Links

* [Hailo Model Explorer](https://hailo.ai/products/hailo-software/model-explorer/)
* [Hailo Model Zoo](https://github.com/hailo-ai/hailo_model_zoo/tree/master/hailo_model_zoo)
* [TAPPAS Post-Processing Libraries](https://github.com/hailo-ai/tappas/tree/master/core/hailo/libs/postprocesses)
* [Camera software](https://www.raspberrypi.com/documentation/computers/camera_software.html)
* [HailoRT Documentation](https://hailo.ai/developer-zone/documentation/hailort-v4-19-0/)
