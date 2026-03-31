---
title: AOSP:Build
date: 2025-10-20
categories:
  - My
tags:
  - aosp
---

# Build AOSP

List of available branches: https://android.googlesource.com/platform/manifest/+refs

* Create a password with [the password generator](https://android.googlesource.com/new-password)
* Clone `main` branch:
```shell
$ repo init --partial-clone -u https://android.googlesource.com/platform/manifest -b main
$ repo sync -c -j12
```
* Checkout to specific branch:
```shell
$ repo forall -c git checkout aosp-main
```

```shell
$ source build/envsetup.sh
$ hmm
... display all available commands ...
$ lunch sdk_car_x86_64-aosp_current-userdebug
$ m
or
$ m <module-name> # Builds only
$ mm # Builds and installs all of the modules in the current directory
$ mmmm <module> # Builds and installs all of the modules in the supplied dir
```

# Build cuttlefish

Build emulator:
```shell
$ sudo apt install -y \
 git devscripts equivs config-package-dev \
 debhelper-compat golang curl
$ git clone https://github.com/google/android-cuttlefish
$ cd android-cuttlefish
$ tools/buildutils/build_packages.sh
$ sudo dpkg -i ./cuttlefish-base_*_*64.deb || sudo apt-get install -f
$ sudo dpkg -i ./cuttlefish-user_*_*64.deb || sudo apt-get install -f
$ sudo usermod -aG kvm,cvdnetwork,render $USER
$ sudo reboot
```

Start emulator:
```shell
$ cd android/aosp/out
$ export TOP=$(pwd)
$ HOME=$TOP/host/linux-x86 $TOP/host/linux-x86/bin/launch_cvd \
 -system_image_dir=$TOP/target/product/vsoc_x86_64_only \
 -console=true \
 -cpus=4 \
 -memory_mb=8192
# Open https://localhost:8443 to interact with UI
```

Stop emulator:
```shell
$ HOME=$TOP/host/linux-x86 $TOP/host/linux-x86/bin/stop_cvd
```

Build image to run in emulator:
```shell
$ source build/envsetup.sh
$ lunch aosp_cf_x86_64_phone-trunk_staging-userdebug
or
$ lunch aosp_cf_x86_64_auto-trunk_staging-userdebug

$ m clean
$ m nothing
$ m
$ emulator -no-snapshot
```

Run adb shell:
```shell
$ $TOP/host/linux-x86/bin/adb devices
List of devices attached
0.0.0.0:6520	device
$ $TOP/host/linux-x86/bin/adb shell
```

Show logs:
```shell
$ screen $TOP/host/linux-x86/cuttlefish/instances/cvd-1/console
$ glogg $TOP/host/linux-x86/cuttlefish/instances/cvd-1/kernel.log
$ glogg $TOP/host/linux-x86/cuttlefish/instances/cvd-1/logs/logcat
$ glogg $TOP/host/linux-x86/cuttlefish/instances/cvd-1/logs/launcher.log
```


