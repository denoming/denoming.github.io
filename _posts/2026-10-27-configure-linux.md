---
title: "Configure Linux"
date: 2022-10-15
categories: [Engineering,Linux]
tags: [ntp]
---

This article is a bunch of small configuration snippets

## Mount USB stick permanently

```
Get UUID of USB disk
$ lsblk -o NAME,FSTYPE,UUID,MOUNTPOINTS
...
sdb                                                                             
└─sdb1                       exfat       7ADB-39AB  
```

Add mounting to `/etc/fstab` config file:
```
$ mkdir /mnt/data
$ sudo nano /etc/fstab
UUID=7ADB-39AB /mnt/data exfat defaults,noatime 0 2
$ sudo mount -a
```

## Install and configure NTP (timesyncd)
```
$ sudo apt update
$ sudo apt install systemd-timesyncd
$ sudo systemctl enable --now systemd-timesyncd
$ timedatectl status
# Should be
System clock synchronized: yes 
NTP service: active
# If not
$ sudo timedatectl set-ntp true
$ sudo systemctl enable --now systemd-timesyncd
```
