---
title: "CMake cheatsheet"
date: 2021-12-01
categories: [Development,CMake]
tags: [cmake]
---

# Command line

## General

List available modules:

```sh
$ cmake --help-module-list
```

List available presents:

```sh
$ cmake --list-presets
# or
$ cmake --build --list-presets
```

Use preset:

```sh
$ cmake --preset "name"
# or
$ cmake --build --preset "name"
```

## Install mode

Install `ProjectRuntime` at specific place using given build directory in `Debug` config mode:

```bash
$ cmake --install /path/to/build/dir   \
        --prefix /path/to/staging/area \
        --config Debug                 \
        --component ProjectRuntime     \
```

Additional flag can be given in case we want to strip debug information of files before installing:

```bash
cmake --install /path/to/build/dir   \
      --prefix /path/to/staging/area \
      --strip
```

# Useful approaches

## Activate C++17 standard in cmake

**Project** specific configuration of C++ standard, works for all targets that will be defined.

```cmake
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
```

**Target** specific configuration (recommended):

```cmake
target_compile_features(myTarget PUBLIC cxx_std_17)
```

or

```cmake
set_target_properties(myTarget PROPERTIES
    CXX_STANDARD 17
    CXX_STANDARD_REQUIRED YES
    CXX_EXTENSIONS NO
)
```

Also there is additional approach to enable only specific feature by `target_compile_features()`:

```cmake
target_compile_features(myTarget
    PUBLIC
        cxx_variadic_templates
        cxx_nullptr
    PRIVATE
        cxx_lambdas
)
```

## Specify RPATH

Get relative path from executables directory to the libraries directories:

```cmake
include(GNUInstallDirs)
file(RELATIVE_PATH relativePath
    ${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_INSTALL_BINDIR}
    ${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}
)
```

Set RPATH for all targets defined after the above line:

```cmake
set(CMAKE_INSTALL_RPATH $ORIGIN $ORIGIN/${relativePath})
```

or set upon particular target:

```cmake
set_target_properties(myTarget
    PROPERTIES
        INSTALL_RPATH "$ORIGIN;$ORIGIN/${relativePath}")
```

## Specify default install prefix path

```cmake
if (NOT WIN32 AND CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
    # Set default install prefix path
    if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
        set(CMAKE_INSTALL_PREFIX "/opt/denoming/${PROJECT_NAME}" CACHE PATH "..." FORCE)
    endif()
endif()
```

## Add project version

```cmake
# Specify project version number.
set (App_VERSION_MAJOR 1)
set (App_VERSION_MINOR 0)

# ...
project (App VERSION "${App_VERSION_MAJOR}.${App_VERSION_MINOR}")
# ...

# ...
configure_file (
  "${PROJECT_SOURCE_DIR}/AppConfig.h.in"
  "${PROJECT_BINARY_DIR}/AppConfig.h"
  )
# ...

include_directories("${PROJECT_BINARY_DIR}")
```

Config file template:

```
#define App_VERSION_MAJOR @App_VERSION_MAJOR@
#define App_VERSION_MINOR @App_VERSION_MINOR@
```

Using in source code:

```cpp
#include <iostream>

#include "AppConfig.h"

int main (int argc, char *argv[])
{
  std::cout << "Major: " << App_VERSION_MAJOR
            << "Minor: " << App_VERSION_MINOR
            << std::endl;
  return 0;
}
```

## Define macro variable

```cmake
# ...
add_definitions(-DSOME_IMPORTANT_DEFINITION)
# ...
```

Using of macro variable in source code:

```cpp
#ifdef SOME_IMPORTANT_DEFINITION
# ... some staff ...
#endif
```

## Enable dependencies optimization

Setting flag below enables dependency optimization of static and object libraries:

```cmake
set(CMAKE_OPTIMIZE_DEPENDENCIES ON)
```