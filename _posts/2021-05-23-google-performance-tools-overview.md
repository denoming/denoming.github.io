---
title: "Google performance tools Overview"
date: 2021-05-23
categories: [Development,Tools]
tags: [memory,gperftools]
---

# Install

```bash
$ sudo apt install libunwind-dev
$ git clone git@github.com:gperftools/gperftools.git
$ cd gperftools
$ cmake -B build -S . -DCMAKE_INSTALL_PREFIX=$HOME/.local
$ cmake --build build --parallel
$ cmake --build build --target install
```

## Install pprof

[Pprof](https://github.com/google/pprof/blob/master/doc/README.md) is a tool that used to work with profiles.

```bash
$ curl -O https://storage.googleapis.com/golang/go1.16.4.linux-amd64.tar.gz
$ sudo tar xf go1.16.4.linux-amd64.tar.gz -C /usr/local
$ mkdir $HOME/.local/go
$ tee -a $HOME/.profile << 'EOF'
# Specify Go path
export GOPATH=$HOME/.local/go
export PATH=$PATH:/usr/local/go/bin:$GOPATH/bin
EOF
$ go version
go version go1.16.4 linux/amd64
$ go get -u github.com/google/pprof
```

## Install Gv (optional)

Gv is a tool that used to visualize artifacts of performance analyzing.
```
$ sudo apt update
$ sudo apt install gv
```

# Use

## Use tcmalloc

Overall documentation about tcmalloc and how it works you can find at `docs/tcmalloc.html` file in repository.

### Heap checker

Documenation available at `docs/heap_checker.html` file in repository.
To use heap checker there are to opportunity:
- by `LD_PRELOAD`:
```bash
$ LD_PRELOAD=$HOME/.local/lib/libtcmalloc.so HEAPCHECK=normal ./example.x
```
Linking with any library **not required**.
- by linking tcmalloc library:
```bash
$ HEAPCHECK=normal ./example.x
```
Use [AddGPerfTools.cmake](https://github.com/karz0n/warehouse/blob/master/cmake/modules/AddGPerfTools.cmake) cmake module and link binary with `GPerfTools::TCMalloc` target.

### Heap profiler

Documenation available at `docs/heapprofile.html` file in repository.
To use heap profiler there are to opportunity:
- by `LD_PRELOAD`:
```bash
$ LD_PRELOAD=$HOME/.local/lib/libtcmalloc.so HEAPPROFILE=./example ./example.x
```
Linking with any library **not required**.
- by linking tcmalloc library:
```bash
$ HEAPPROFILE=./example ./example.x
$ pprof -gv ./example.x ./example.XXXX.heap
or
$ pprof -pdf ./example.x ./example.XXXX.heap
```
Use [AddGPerfTools.cmake](https://github.com/karz0n/warehouse/blob/master/cmake/modules/AddGPerfTools.cmake) cmake module and link binary with `GPerfTools::TCMalloc` target.

### CPU profiler

Documenation available at `docs/cpuprofile.html` and `docs/cpuprofile-fileformat.html`  files in repository.
To use CPU profiler you need:
- link binary with `GPerfTools::Profiler` target
- use `ProfilerStart(<name>)` and `ProfilerStop()` to designate certain block of code.
- run binary as: `CPUPROFILE=./example.out ./example.x`

To visualize profile:
```bash
$ pprof --gv ./example.x ./example.out
or
$ pprof --pdf ./example.x ./example.out
```

Use [AddGPerfTools.cmake](https://github.com/karz0n/warehouse/blob/master/cmake/modules/AddGPerfTools.cmake) cmake module to find certain packages.

**Note:** To link binary with tcmalloc and profile libraries you have to link with `GPerfTools::TCMallocAndProfiler` target.