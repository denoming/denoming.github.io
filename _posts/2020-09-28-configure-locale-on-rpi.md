---
title: "Configure locale on Raspberry Pi OS"
date: 2020-09-06
categories: [Engineering,RaspberryPi]
tags: [rpi,linux]
---

1. Update locale:
```bash
$ sudo raspi-config # Choose: 4 Localisation Options => I1 Change Locale
$ sudo update-locale \
LANGUAGE=en_US.UTF-8 \
LANG=en_US.UTF-8 \
LC_TYPE=en_US.UTF-8 \
LC_ALL=en_US.UTF-8
```
2. Reboot device;
3. Generate locale:
```bash
$ sudo locale-gen en_US.UTF-8
$ export LANGUAGE=en_US.UTF-8
$ export LANG=en_US.UTF-8
$ export LC_TYPE=en_US.UTF-8
$ export LC_ALL=en_US.UTF-8
```
4. Configure locale at environment file:
```bash
sudo tee /etc/environment > /dev/null << EOF
export LANGUAGE=en_US.UTF-8
export LANG=en_US.UTF-8
export LC_TYPE=en_US.UTF-8
export LC_ALL=en_US.UTF-8
EOF
```