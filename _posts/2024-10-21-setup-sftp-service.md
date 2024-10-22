---
title: "Setup SFTP service"
date: 2024-10-21
categories: [Engineering,Linux]
tags: [sftp]
---

Install packages:
```shell
$ sudo apt update
$ sudo apt install openssh-server
```

Check service status:
```shell
$ sudo systemctl status ssh
```

Create service group and user:
```shell
$ sudo groupadd sftp
$ sudo useradd -m -g sftp -s /bin/false sftp
$ sudo passwd sftp
```

Configre:
```shell
$ sudo vim /etc/ssh/sshd_config
...

Subsystem sftp internal-sftp

Match Group sftp
    X11Forwarding no
    AllowTcpForwarding no
    ChrootDirectory /data/users/%u
    ForceCommand internal-sftp
```

Create root dir:
```shell
$ sudo mkdir -p /data/sftp
$ sudo chown root:sftp /data/sftp
```

Create user dir:
```shell
$ sudo mkdir -p /data/sftp/sftp
$ sudo chmod 755 /data/sftp/sftp
$ sudo chown root:sftp /data/sftp/sftp
```

**Note: All dirs must be root owned "root:sftp" and have 755 access mode.**