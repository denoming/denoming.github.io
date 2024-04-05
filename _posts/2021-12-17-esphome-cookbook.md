---
title: "ESPHome cookbook"
date: 2021-08-28
categories: [Engineering,ESP]
tags: [esp32,esp01,esp8266]
---

# Setup

```bash
$ sudo apt update
$ sudo apt install python3.9 python3-pip python-is-python3
$ sudo pip3 install -U \
-f https://extras.wxpython.org/wxPython4/extras/linux/gtk3/ubuntu-20.04 \
wxPython
$ sudo pip3 install esptool requests esphomeflasher
```

# Flash

```bash
$ esphomeflasher -p /dev/ttyUSB0 --esp32 sensority01.bin
```
