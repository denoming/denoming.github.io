---
title: "Vcpkg overview"
date: 2023-09-16
categories: [Engineering,Linux]
tags: [vcpkg,tools,package-manager]
---

# Installing

Clone repository to some folder:
```
$ cd <path-to-folder>
$ git clone https://github.com/microsoft/vcpkg
$ cd vcpkg
$ ./vcpkg/bootstrap-vcpkg.sh
```
Futher `VCPKG_ROOT` mention will mean the path to cloned vcpkg repository.

Make a symlink to vcpkg tool:
```
$ ln -fs $VCPKG_ROOT/vcpkg $HOME/.local/bin/vcpkg 
```

# Using

## Classic mode

```
$ vcpkg search sqlite
libodb-sqlite        2.4.0            Sqlite support for the ODB ORM library
sqlite3              3.32.1           SQLite is a software library that implements a se...
$ vcpkg install sqlite3
$ vcpkg list
sqlite3:x86-windows         3.32.1           SQLite is a software library that implements a se...
```

## Manifest mode

```shell
$ vcpkg new --application --name <project-name> --version <project-version>
```
or create following files at the root of project manually:
* [vcpkg.json](https://learn.microsoft.com/en-us/vcpkg/reference/vcpkg-json)
* [vcpkg-configuration.json](https://learn.microsoft.com/en-us/vcpkg/reference/vcpkg-configuration-json) (optional)

### CMake

Integration with CMake requires using certain toolchain file:
```
cmake -B build -S . -DCMAKE_TOOLCHAIN_FILE=$VCPKG_ROOT\vcpkg\scripts\buildsystems\vcpkg.cmake
...
cmake --build build --parallel
```

# Porting

```
vcpkg-registry
    ports
        package
            portfile.cmake - describes hot to build and install the package
            vcpkg.json - describes the package's metadata and dependencies
versions
    j-
        package.json
    baseline.json - contains the "latest versions" at a certain commit
```

* Update `ports/package.cmake` file if needed
* Check new version by installing it:
```
$ cd <path-to-registry>
$ vcpkg install package --overlay-ports=ports
```
* get the git tree ID of the directory:
```
$ git add ports/package
$ git commit -m "..."
$ git rev-parse HEAD:ports/package
57116c5862aeae410fba08fcd54f90d8be714c17
```
* Update `versions/j-/jarvisto.json` file: add new version and certain revision ID
* Update baseline version at `baseline.json` (put the lastest version)
* Commit changes by amend commit:
```
$ git add --all
$ git commit --amend
```

# Integration

## CMake

### Configure custom triple

```text
set(VCPKG_TARGET_TRIPLET "x64-linux-dynamic" CACHE STRING "Target triplet configuration")
set(VCPKG_HOST_TRIPLET "x64-linux")
```

### Change feature list

Update `vcpkg.json` file:
```text
  "features": {
    "feature1": {
      "description": "Feature1 supporting",
      "dependencies": [
        {
          "name": "libsndfile",
          "version>=": "1.2.0",
          "default-features": false
        }
      ]
    }
  }
```

Update `CMakeLists.txt` file:
```text
if (ENABLE_FEATURE1)
    list(APPEND VCPKG_MANIFEST_FEATURES "feature1")
endif()
```

### Configure custom registry

Create `vcpkg-configuration.json` with following content:
```json
{
  "default-registry": {
    "kind": "git",
    "baseline": "9d47b24eacbd1cd94f139457ef6cd35e5d92cc84",
    "repository": "https://github.com/microsoft/vcpkg"
  },
  "registries": [
    {
      "kind": "git",
      "baseline": "5cc830699bc3510b9264032d8d993fb25abc4701",
      "repository": "git@github.com:karz0n/vcpkg-registry.git",
      "packages": [ "package1", "package1" ]
    }
  ]
}
```

Where `packages` is the list of packages which should be installed from custom registry.