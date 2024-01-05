---
title: "Setup network-wide ad-blocker based on RaspberryPi"
date: 2020-09-06
categories: [Engineering,RaspberryPi]
tags: [rpi,linux]
---

1. Visit [side](https://www.raspberrypi.org/downloads) and download `Imager` tools;
2. Install `Imager` tool:

```shell
$ sudo apt install ./imager_<version>_amd64.deb
```

3. Run tool and write any RaspberryPi OS to the SD card;
4. Plug-in SD card to RaspberryPi and log-in to the device;
5. Configure device by `raspi-config` tool and restart;
6. Log-in to the device, run installation and follow instructions:

```shell
$ sudo curl -sSL https://install.pi-hole.net | bash
```

7. Change auto generated admin password by command:

```shell
$ pihole -a -p
```

8. Enjoy.

Note: To get access of web-admin page use link: http://<ip-address>/admin
