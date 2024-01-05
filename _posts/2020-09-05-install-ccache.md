---
title: "Install ccache"
date: 2020-09-05
categories: [Development,Tools]
tags: [cpp,ccache]
---

## Install

```bash
$ sudo apt install -y ccache
```

## Configure

To enable unlimited cache size:
```bash
$ ccache -F 0
$ ccache -M 0
```

To show cache statistics:
```bash
$ ccache -s
```

To empty cache:
```bash
$ ccache -Cz
```

## Use

Using _ccache_ in cmake project required append one cmake module: [EnableCcache.cmake](https://github.com/karz0n/cmake-modules/blob/master/EnableCcache.cmake). Place this file into project at the modules location directory and include _EnableCcache.cmake_ it:
```bash
include(EnableCcache)
```
