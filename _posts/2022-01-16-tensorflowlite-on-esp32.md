---
title: "Using TensorFlow Lite on ESP32"
date: 2022-01-23
categories: [Engineering,Embedded]
tags: [tensorflow,esp32,esp-idf]
---

# Install ESP-IDF

```shell
$ sudo apt-get install git wget flex bison gperf unzip curl \
       python3 python3-pip python3-setuptools \
       cmake ninja-build ccache \
       libffi-dev libssl-dev dfu-util libusb-1.0-0
$ easy_install Pillow Image Wave
```

```shell
$ mkdir -p $HOME/.local/opt/esp
$ cd $HOME/.local/opt/esp
$ git clone --recursive https://github.com/espressif/esp-idf.git
$ cd esp-idf
$ git checkout -b v4.4 v4.4
$ git submodule update --init --recursive
$ bash install.sh esp32
$ tee -a $HOME/.zprofile > /dev/null <<'EOF'
alias get_idf='. $HOME/.local/opt/esp/esp-idf/export.sh'
EOF
$ source ~/.zprofile
```

# Build

Fetch components and build examples:

```shell
$ git clone --recursive https://github.com/espressif/tflite-micro-esp-examples.git
$ cd tflite-micro-esp-examples/examples/hello_world
$ get_idf
$ idf.py set-target esp32
$ idf.py build
```

Update components (optional):

```shell
$ cd tflite-micro-esp-examples/scripts
$ bash sync_from_tflite_micro.sh
```

There are two ESP-IDF components at `components` sub-folder ready to use:

* tflite-lib
* esp-nn
