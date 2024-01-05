---
title: "Using PlatformIO"
date: 2021-05-23
categories: [Engineering,Embedded]
tags: [embedded,platformio,pio]
---

# Install

## Platform CLI

Documentation is available at - [official site](https://docs.platformio.org/en/latest/core/installation/methods/installer-script.html).

Install PlatformIO Core
```shell
$ python3 -c "$(curl -fsSL https://raw.githubusercontent.com/platformio/platformio/master/scripts/get-platformio.py)"
```
Set bin directory in PATH env:
```shell
$ tee -a ~/.zshenv > /dev/null <<EOF

# Add PlatformIO bin dir to the PATH
export PATH=${PATH}:${HOME}/.platformio/penv/bin
EOF
$ source ${HOME}/.profile
```

## Platform IDE

* Install VS Code
* Install "PlatformIO IDE" extension in VS Code

# Configure

```sh
$ curl -fsSL https://raw.githubusercontent.com/platformio/platformio-core/develop/platformio/assets/system/99-platformio-udev.rules | sudo tee /etc/udev/rules.d/99-platformio-udev.rules
$ sudo service udev restart
$ sudo usermod -a -G dialout $USER
$ sudo usermod -a -G plugdev $USER
<REBOOT>
```

__Note:__ Check `99-platformio-udev.rules` file that it should contain your board's PID and VID (to list devices use `pid device list` command)

# IDE

## Visual Studio Code

Plugins:

* PlatformIO IDE
* CMake
* ...

# Tools

## Picocom

Picocom is a minimal dumb-terminal emulation program that is great for accessing a serial port based Linux console.

Install:
```sh
$ sudo apt install picocom

```

Use:
```sh
$ picocom -b 115200 -r -l /dev/ttyUSB0
```

# Tips

To show output on COM port from connected Arduino:
```bash
$ pio device monitor -e uno
```

To install some library globally:
```bash
$ pio lib --global install "<somePath>/<someName>"
```



