---
title: "Overclock Raspberry Pi 3"
date: 2024-04-01
categories: [Engineering,RaspberryPi]
tags: [rpi,linux]
---

Getting current CPU frequency:
```shell
$ cat /sys/devices/system/cpu/cpu0/cpufreq/cpuinfo_min_freq
$ cat /sys/devices/system/cpu/cpu0/cpufreq/cpuinfo_max_freq
$ cat /sys/devices/system/cpu/cpu0/cpufreq/cpuinfo_cur_freq
```

* `cpuinfo_cur_freq` is a current running frequency of RPI

Monitoring the CPU temperature:
```shell
$ while true ; do vcgencmd measure_temp ; sleep 1 ; done
```

Benchmarking:
```shell
$ sysbench --test=memory --cpu-max-prime=2000 --num-threads=4 run
```

# Overclock

## By config file

Parameters should be set in `/boot/config.txt` file.

First enable following parameters (lite overclocking):
```text
force_turbo=1
boot_delay=1
```

Start from following values:
```
arm_freq=1200
core_freq=400
gpu_freq=400
sdram_freq=450
over_voltage_sdram=0
```

Then following values:
```
arm_freq=1300
core_freq=500
gpu_freq=500
sdram_freq=500
over_voltage_sdram=0
```

If Pi is stable `arm_freq` can be increased up to 1500.
If Pi become unstable try to increase `over_voltage_sdram` up to 4-5, step by step.
If it doesn't help you need to reduce `arm_freq`.

## By Yocto

First enable following parameters:
```
RPI_EXTRA_CONFIG = ' \n \
    force_turbo=1 \n \
    boot_delay=1 \n \
    '
```

Defaults:
```
ARM_FREQ = "1200"
GPU_FREQ = "250"
CORE_FREQ = "400"
SDRAM_FREQ = "450"
OVER_VOLTAGE = "0"
```

Then following values:
```
ARM_FREQ = "1300"
CORE_FREQ = "500"
GPU_FREQ = "500"
SDRAM_FREQ = "500"
OVER_VOLTAGE = "4"
```



