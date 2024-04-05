---
title: "Build and run ARM docker container on x86"
date: 2020-09-05
categories: [Engineering,Docker]
tags: [docker,arm,x86]
---

## Prerequisites

Install QEMU:
```bash
$ sudo apt update
$ sudo apt install -y qemu qemu-user-static qemu-user binfmt-support
```

Register QEMU in the build agent:
```bash
$ docker run --rm --privileged multiarch/qemu-user-static:register --reset
```

## Build

```bash
$ mkdir -p base/aarch64v8
$ cd base/aarch64v8
$ cp /usr/bin/qemu-aarch64-static .
$ cat << EOF > Dockerfile
FROM arm64v8/ubuntu:18.04
COPY ./qemu-aarch64-static /usr/bin/qemu-aarch64-static
EOF
$ docker build -f ./Dockerfile -t my/base:aarch64v8 .
```

There are options of base system:
* [arm32v6](https://hub.docker.com/u/arm32v6/)
* [arm32v7](https://hub.docker.com/u/arm32v7/)
* [s390x](https://hub.docker.com/r/s390x/)
* [ppc64le](https://hub.docker.com/u/ppc64le/)

QEMU for particular system located at _/usr/bin_ and prefixed by `qemu-`.

## Run

```bash
$ docker run -it --rm my/base:aarch64v8 /bin/bash
```
