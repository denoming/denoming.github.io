---
title: "How to build POCO library"
date: 2020-04-27
categories: [Development,Bootstrap]
tags: [poco]
---

# Prerequisites

## OpenSSL

```bash
$ cd <dir>
$ export TOP_DIR=$PWD
$ wget https://www.openssl.org/source/openssl-1.1.1d.tar.gz
$ tar -xf openssl-1.1.1d.tar.gz && rm openssl-1.1.1d.tar.gz
$ cd openssl-1.1.1d
$ ./config shared \
--prefix=$TOP_DIR/openssl \
--openssldir=$TOP_DIR/openssl > /dev/null
$ make -j${nproc}
$ make install
$ cd $TOP_DIR && rm -fr openssl-1.1.1d # Optional
```

# Build

```bash
$ cd $TOP_DIR
$ git clone -b poco-1.10.1 https://github.com/pocoproject/poco.git poco-src
$ cd poco-src
$ mkdir -p cmake-build && cd cmake-build
$ cmake -Wno-dev -Wno-deprecated \
-DENABLE_MONGODB=OFF \
-DENABLE_REDIS=OFF \
-DENABLE_DATA=OFF \
-DENABLE_DATA_SQLITE=OFF \
-DENABLE_DATA_MYSQL=OFF \
-DENABLE_DATA_ODBC=OFF \
-DENABLE_ZIP=OFF \
-DENABLE_PAGECOMPILER=OFF \
-DENABLE_PAGECOMPILER_FILE2PAGE=OFF \
-DOPENSSL_ROOT_DIR=$TOP_DIR/openssl \
-DOPENSSL_USE_STATIC_LIBS=TRUE \
-DCMAKE_INSTALL_RPATH=$TOP_DIR/poco/lib \
-DCMAKE_INSTALL_PREFIX=$TOP_DIR/poco ..
$ cmake --build . --target install
$ cd $TOP_DIR && rm -fr poco-src # Optional
```

**Note:** Pay attention at `CMAKE_INSTALL_RPATH`. We correct RPATH for libraries. You can skip it and in that case RPATH will be empty.

# Use

```cmake
list(APPEND CMAKE_PREFIX_PATH "${PROJECT_SOURCE_DIR}/thirdparty/poco")

find_package(Poco
    REQUIRED
    CONFIG
    COMPONENTS Util JSON XML Net Foundation
)

if(${Poco_FOUND})
    message(STATUS "Using Poco ${Poco_VERSION} library")
else()
    message(FATAL_ERROR "Poco library NOT found")
endif()
```

