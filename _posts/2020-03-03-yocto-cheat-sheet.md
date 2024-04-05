---
title: "Yocto cheat sheet"
date: 2020-03-01
categories: [Engineering,Yocto]
tags: [yocto,bitbake]
---

# Debug

Debug failing recipe:
* List all recipes and identify the recipe:
```
$ bitbake-layers show-recipes
```
* Find any recipe or append files related to the package:
```shell
$ find .. -name "*<name>*.bb*"
```
* List all taks of particular recipe:
```
$ bitbake -c listtasks <recipe>
```
* Reproduce an error:
```shell
$ bitbake -c clean <recipe> && bitbake <recipe>
```
* Dumping environment:
```
$ bitbake -e | less
and
$ bitbake -e <recipe> | less
(bitbake -e <recipe> | grep ^$ to locate source dir)
(bitbake -e <recipe> | grep ^WORKDIR to locate work dir)
```
* Reading the task log:
```
$ tmp/work/<arch>-poky-linux/<recipe>/<version>/temp
```
* Add more logs by editing `meta/classes/logging.bbclass` file:
```
# To log from python:
bb.plain -> none; Output: logs console
bb.note -> logger.info; Output: logs
bb.warn -> logger.warning; Output: logs console
bb.error -> logger.error; Output: logs console
bb.fatal -> logger.critical; Output: logs console
bb.debug -> logger.debug; Output: logs console
# To log from shell:
bbplain -> Prints exactly what is passed in. Use sparingly.
bbnote -> Prints noteworthy conditions with the NOTE prefix.
bbwarn -> Prints a non-fatal warning with the WARNING prefix.
bberror -> Prints a non-fatal error with the ERROR prefix.
bbfatal -> Prints a fatal error and halts the build.
bbdebug -> Prints debug messages depending on log level.
# Usage: bbdebug 1 "first level debug message"
#        bbdebug 2 "second level debug message
```
* Build of recipe in devshell:
```
$ bitbake -c devshell <recipe>
- Fetch source code of recipe and perform patchinh if needed
- Set all environment variable
- Enter into shell (configure, make or $CC are available)
(perfect to experiment with CFLAGS or LDFLAGS)
```
* Get dependency graph:
```
$ bitbake -v <recipe>
or
$ bitbake -f <recipe> -u taskexp
```

# Commands

## Basic

Check if certain package is present on current setup:
~~~
bitbake -s | grep <package>
~~~

List avaialble tasks:
```shell
$ bitbake -c listtasks <recipe-name>
or
$ bitbake -c listtasks core-image-minimal
```

Run task for a target and all its dependencies:
```shell
$ bitbake <image-name> --runall=fetch
```

Show possible image to bake:
~~~
$ bitbake-layers show-recipes "*-image-*"
or
$ cd sources/poky
$ ls meta*/recipes*/images/*.bb
~~~

Show possible layers:
~~~
$ bitbake-layers show-layers
~~~

## Image

Get complete list of packages that will be built for given image:
~~~
bitbake -g <image> && cat pn-buildlist | grep -ve "native" | sort | uniq
~~~

Get docker image from yocto root file system:

```bash
$ docker import - "name-of-image" < "image".tar.bz2
$ docker run -it --rm "name-of-image" bash
```

## SDK

Build SDK for particular image:
```shell
$ bitbake <image-name> -c populate_sdk
```
Build extensible SDK for particular image:
```shell
$ bitbake <image-name> -c populate_sdk_ext
```
Build basic toolchain with just C and C++ cross compilers, libraries and headers:
```shell
$ bitbake meta-toolchain
```

## Package

List of tools:
* **oe-pkgdata-util**
* **oe-pkgdata-browser**

Examples:
* Lists all packages **that have been built**:
~~~
$ oe-pkgdata-util list-pkgs
~~~

* Lists the files and directories contained in the given packages:
~~~
$ oe-pkgdata-util list-pkg-files package
~~~

* Lists the names of the packages that contain the given paths:
~~~
oe-pkgdata-util find-path <path>
~~~
For example, the following tells us that */usr/share/man/man1/make.1* is contained in the make-doc package:
~~~
$ oe-pkgdata-util find-path /usr/share/man/man1/make.1
make-doc: /usr/share/man/man1/make.1
~~~

* Lists the name of the recipes that produce the given packages:
~~~
oe-pkgdata-util lookup-recipe <package>
~~~

* Get information of particular package:
~~~
$ oe-pkgdata-util read-value <value> <pkg1>
~~~
Where _value_ can be: SUMMARY, DESCRIPTION, etc.

* View dependency information:
~~~
$ bitbake -g <recipe> -u depexp
~~~

## Recipe

List of tools:
* __recipetool__
* __bitbake-layers__

Examples:

* Get a list of recipes: 
```
$ bitbake-layers show-recipes
```

## Task

* Get a list of tasks:
```
$ bitbake -c listtasks <recipe>
```
* Run particular task:
```
$ bitbake -c <task-name> <recipe>
# do_<task-name> (e.g. fetch, configure, compile)
```
* Clean output files (unpack, configure, conpile, install and package artifacts):
```
$ bitbake -c clean <recipe>
```
* Clean all files (output files, shared state cache and downloaded source files):
```
$ bitbake -c cleanall <recipe>
```
* Remote all output files and shared state cache:
```
$ bitbake -c cleansstate <recipe>
```
