---
title: Build and run ARM docker container
date: 2020-09-05
categories:
  - Engineering
  - Docker
tags:
  - docker
  - arm
  - x86
  - ArchLinux
---
Before installing user level emulation:
```shell
$ docker run --rm --platform linux/arm64 -t arm64v8/ubuntu uname -m
exec format error
```

Installing:
```shell
$ sudo pamac install --no-confirm \
qemu-user-static qemu-user-static-binfmt
```

File `/usr/bin/qemu-aarch64-static` as well as `F` flag must be present:
```shell
$ ls -la /usr/bin/qemu-aarch64-static
-rwxr-xr-x 1 root root 6067560 Sep  6 16:46 /usr/bin/qemu-aarch64-static
$ cat /proc/sys/fs/binfmt_misc/qemu-aarch64
enabled
interpreter /usr/bin/qemu-aarch64-static
flags: PF
offset 0
magic 7f454c460201010000000000000000000200b700
mask ffffffffffffff00fffffffffffffffffeffffff
```

Run docker:
```shell
# Run 
$ docker run --rm --platform linux/arm64 -t arm64v8/debian:bookworm uname -m
aarch64
```

