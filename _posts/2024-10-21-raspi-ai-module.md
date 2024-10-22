---
title: "Install and user Raspberry Pi 5 AI extension"
date: 2024-10-21
categories: [Engineering,Linux]
tags: [raspberry,ai]
---

Upgrade OS:
```shell
$ sudo apt update
$ sudo apt full-upgrade
$ sudo rpi-eeprom-update
BOOTLOADER: up to date
   CURRENT: Mon 23 Sep 13:02:56 UTC 2024 (1727096576)
    LATEST: Mon 23 Sep 13:02:56 UTC 2024 (1727096576)
   RELEASE: default (/lib/firmware/raspberrypi/bootloader-2712/default)
            Use raspi-config to change the release.

If you see 6 December 2023 or a later date, proceed to the next step
If not you need to update by the following command:
$ sudo rpi-eeprom-update -a
```

Install `hailo-all` package:
```shell
$ sudo apt install hailo-all
$ sudo reboot
```

Check module version:
```shell
$ sudo hailortcli fw-control identify
Executing on device: 0000:01:00.0
Identifying board
Control Protocol Version: 2
Firmware Version: 4.18.0 (release,app,extended context switch buffer)
Logger Version: 0
Board Name: Hailo-8
Device Architecture: HAILO8L
Serial Number: HLDDLBB241601754
Part Number: HM21LB1C2LAE
Product Name: HAILO-8L AI ACC M.2 B+M KEY MODULE EXT TMP
```

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
