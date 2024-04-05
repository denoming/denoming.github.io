---
title: "ESP-01 from scratch"
date: 2021-07-17
categories: [Engineering,ESP]
tags: [esp01]
---

# Overview

## Specification

* 11 b/g/n protocol
* Wi-Fi Direct (P2P), soft-AP
* Integrated TCP/IP protocol stack
* Built-in low-power 32-bit CPU
* SDIO 2.0, SPI, UART

## Pin-out

![pinout]({{site.utl}}/assets/img/posts/ESP-01-ESP8266-pinout.png)

# Useful

## Change baud-rate

To update baud-rate use the next command:

```text
    AT+UART_DEF=<baudrate>,<databits>,<stopbits>,<parity>,<flow control>
```

Example (change baud-rate to 9600 bps):

```bash
$ pio device list
...
/dev/ttyUSB0
...
$ pio device monitor -p /dev/ttyUSB0 -b 115200 --rtscts
AT+UART_DEF=9600,8,1,0,1
OK
<Ctrl+C>
$ pio device monitor -p /dev/ttyUSB0 -b 9600 --rtscts
AT+UART_DEF?
+UART_DEF:9600,8,1,0,1
OK
<Ctrl+C>
```

# Links

* [ESP8266 Arduino Documentation](https://arduino-esp8266.readthedocs.io/en/latest/#)
* [Upload AT firmware to ESP8266]({% post_url 2021-07-17-upload-at-firmware-to-esp01 %})
* [Configure Arduino IDE to support ESP8266]({% post_url 2021-07-17-configure-arduinoide-to-support-esp01 %})
