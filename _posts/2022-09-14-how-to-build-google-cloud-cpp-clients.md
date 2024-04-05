---
title: "How to build Google Cloud clients for C++"
date: 2022-09-14
categories: [Development,Bootstrap]
tags: [grpc]
---

# Prerequisites

Install base packages:
```shell
$ export DEBIAN_FRONTEND=noninteractive
$ sudo apt-get update && \
$ sudo apt-get --no-install-recommends install -y apt-transport-https apt-utils \
        automake build-essential ccache cmake ca-certificates curl git \
        gcc g++ libc-ares-dev libc-ares2 libcurl4-openssl-dev libre2-dev \
        libssl-dev m4 make pkg-config tar wget autoconf zlib1g-dev
```

# Build

## Build abseil

By default abseil ABI implies C++>=17 is enabled. Installing abseil with the default configuration is error-prone, unless you can guarantee that all the code using abseil (gRPC, google-cloud-cpp) is compiled with the same C++ version. Build abseil with switching the default configuration to pin abseil ABI to the version used at compile time:

```shell
$ mkdir -p $HOME/source/abseil-cpp && cd $HOME/source/abseil-cpp
$ curl -sSL https://github.com/abseil/abseil-cpp/archive/20220623.1.tar.gz | \
  tar -xzf - --strip-components=1 && \
  sed -i 's/^#define ABSL_OPTION_USE_\(.*\) 2/#define ABSL_OPTION_USE_\1 0/' "absl/base/options.h"
$ cmake -Bcmake-out \
-DCMAKE_BUILD_TYPE=Release \
-DCMAKE_INSTALL_PREFIX=$HOME/.local \
-DABSL_BUILD_TESTING=OFF \
-DBUILD_SHARED_LIBS=yes
$ cmake --build cmake-out --parallel
$ cmake --build cmake-out --target install
```

## Build protobuf

Build protobuf:
```shell
$ mkdir -p $HOME/source/protobuf && cd $HOME/source/protobuf
$ curl -sSL https://github.com/protocolbuffers/protobuf/archive/v21.7.tar.gz | \
  tar -xzf - --strip-components=1
$ cmake -Bcmake-out \
-DCMAKE_PREFIX_PATH=$HOME/.local \
-DCMAKE_BUILD_TYPE=Release \
-DCMAKE_INSTALL_PREFIX=$HOME/.local \
-DBUILD_SHARED_LIBS=yes \
-Dprotobuf_BUILD_TESTS=OFF
$ cmake --build cmake-out --parallel
$ cmake --build cmake-out --target install
```

## Build gRPC

Build gRPC (without examples):
```shell
$ mkdir -p $HOME/source/grpc && cd $HOME/source/grpc
$ curl -sSL https://github.com/grpc/grpc/archive/v1.49.1.tar.gz | \
  tar -xzf - --strip-components=1
$ cmake -Bcmake-out \
-DCMAKE_PREFIX_PATH=$HOME/.local \
-DCMAKE_BUILD_TYPE=Release \
-DCMAKE_INSTALL_PREFIX=$HOME/.local \
-DBUILD_SHARED_LIBS=yes \
-DgRPC_INSTALL=ON \
-DgRPC_BUILD_TESTS=OFF \
-DgRPC_ABSL_PROVIDER=package \
-DgRPC_CARES_PROVIDER=package \
-DgRPC_PROTOBUF_PROVIDER=package \
-DgRPC_RE2_PROVIDER=package \
-DgRPC_SSL_PROVIDER=package \
-DgRPC_ZLIB_PROVIDER=package
$ cmake --build cmake-out --parallel
$ cmake --build cmake-out --target install
```

Note: To build gRPC at least 16Gb RAM and 32Gb swap required.

## Build crc32

```shell
$ mkdir -p $HOME/source/crc32c && cd $HOME/source/crc32c
$ curl -sSL https://github.com/google/crc32c/archive/1.1.2.tar.gz | \
  tar -xzf - --strip-components=1
$ cmake -Bcmake-out \
-DCMAKE_BUILD_TYPE=Release \
-DCMAKE_INSTALL_PREFIX=$HOME/.local \
-DBUILD_SHARED_LIBS=yes \
-DCRC32C_BUILD_TESTS=OFF \
-DCRC32C_BUILD_BENCHMARKS=OFF \
-DCRC32C_USE_GLOG=OFF
$ cmake --build cmake-out --parallel
$ cmake --build cmake-out --target install
```

## Build json

```shell
$ mkdir -p $HOME/source/json && cd $HOME/source/json
$ curl -sSL https://github.com/nlohmann/json/archive/v3.11.2.tar.gz | \
  tar -xzf - --strip-components=1
$ cmake -Bcmake-out \
-DCMAKE_BUILD_TYPE=Release \
-DCMAKE_INSTALL_PREFIX=$HOME/.local \
-DBUILD_SHARED_LIBS=yes \
-DJSON_BuildTests=OFF
$ cmake --build cmake-out --parallel
$ cmake --build cmake-out --target install
```

## Build google cloud C++ client libraries

As example, let's build text-to-speech Google Cloud C++ client library:
```shell
$ mkdir -p $HOME/source/google-cloud-cpp && cd $HOME/source
$ git clone -b v2.2.1 https://github.com/googleapis/google-cloud-cpp.git
$ cd google-cloud-cpp
$ cmake -Bcmake-out \
-DCMAKE_PREFIX_PATH=$HOME/.local \
-DCMAKE_INSTALL_PREFIX=$HOME/.local \
-DBUILD_TESTING=OFF \
-DBUILD_SHARED_LIBS=ON \
-DGOOGLE_CLOUD_CPP_ENABLE=texttospeech \
-DGOOGLE_CLOUD_CPP_ENABLE_EXAMPLES=OFF \
$ LD_LIBRARY_PATH=$HOME/.local/lib cmake --build cmake-out --parallel
$ cmake --build cmake-out --target install
```

Note: Additional Google Cloud C++ client library can be spicified in `GOOGLE_CLOUD_CPP_ENABLE` cmake option.

# Use

The [AddGrpc.cmake](https://github.com/karz0n/cmake-modules/blob/master/AddGrpc.cmake), [FindGrpc.cmake](https://github.com/karz0n/cmake-modules/blob/master/FindGrpc.cmake) and [FindProtobufWithTargets.cmake](https://github.com/karz0n/cmake-modules/blob/master/FindProtobufWithTargets.cmake) modules should be added to project to leverage gRPC capabilities.

# Links

[Git repository of C++ Client Libraries for Google Cloud Services](https://github.com/googleapis/google-cloud-cpp)
[Packaging C++ Client Libraries for Google Cloud Services](https://github.com/googleapis/google-cloud-cpp/blob/main/doc/packaging.md)