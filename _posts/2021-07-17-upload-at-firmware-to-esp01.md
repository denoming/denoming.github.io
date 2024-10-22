---
title: "Upload AT firmware to ESP-01"
date: 2021-07-17
categories: [Engineering,ESP]
tags: [esp01]
---

# Overview

In this post we will consider uploading AT firmware (default firmware) to the ESP-01 (ESP8266 chip) board.

# Download
[Main site](https://www.espressif.com/en/products/socs/esp8266ex/resources) contains huge amount of tools, SDK and documentation. We will need only part of them, namely:

* [ESP8266 NONOS SDK](https://github.com/espressif/ESP8266_NONOS_SDK/releases)
* [Flash Download Tools](https://www.espressif.com/sites/default/files/tools/flash_download_tool_v3.8.8_0.zip) __(only if you work with Windows)__

# Upload

## Under Windows

1. Connect ESP-01 with `USB to ESP-01` adapter, switch on `PROG` mode on adapter;
2. Plug-in to USB port;
3. Run `Flash Download Tools`;
4. Select `SPIDownload` tab;
5. Uncheck all check-boxes;
6. Choose correct `COM` port;
7. Choose `40MHz` in `SPI SPEED`;
8. Click button `START`;
     After that, the relevant information about module will be read:
```text
flash vendor:
D8h : N/A
flash devID:
4014h
QUAD;8Mbit
crystal:
26 Mhz
```
From given above output we can come to the conclusions:
 - EEPROM memory size, 8Mbit (1MB)
 - the frequency of the crystal oscillator, 26MHz
Additionally, we can see the MAC addresses of the module in AP mode:
```text
	AP:   9A-F4-AB-ED-31-B2
 	STA:  98-F4-AB-ED-31-B2
```
8. Choose `40MHz` in `SPI SPEED`, choose `QIO` in `SPI MODE`, choose `8Mbit` in `FLASH SIZE`;
9. Specify paths to the files in accordance with table below:
```text
ESP8266_NONOS_SDK-3.0.4\bin\boot_v1.7.bin					0x00000
ESP8266_NONOS_SDK-3.0.4\bin\at\512+512\user1.1024.new.2.bin 0x01000
ESP8266_NONOS_SDK-3.0.4\bin\blank.bin						0xFB000
ESP8266_NONOS_SDK-3.0.4\bin\esp_init_data_default_v08.bin   0xFC000
ESP8266_NONOS_SDK-3.0.4\bin\blank.bin						0xFE000
ESP8266_NONOS_SDK-3.0.4\bin\blank.bin						0x7E000
```
10. Click `Start` button and wait until progress bar reach the end;
11. Done.

## Under Linux

Install the ESP tool:
```bash
$ pip install esptool
```

Get information from the board:
```bash
$ esptool.py --chip esp8266 read_mac
$ esptool.py --chip esp8266 chip_id
$ esptool.py --chip esp8266 flash_id
...
Detected flash size: 1MB
...
```

Flash the downloaded image:
```bash
$ esptool.py write_flash \
--flash_size 1MB \
0x0 ESP8266_NONOS_SDK-3.0.5/bin/boot_v1.7.bin \
0x01000 ESP8266_NONOS_SDK-3.0.5/bin/at/512+512/user1.1024.new.2.bin \
0xfb000 ESP8266_NONOS_SDK-3.0.5/bin/blank.bin \
0xfc000 ESP8266_NONOS_SDK-3.0.5/bin/esp_init_data_default_v08.bin \
0xfe000 ESP8266_NONOS_SDK-3.0.5/bin/blank.bin \
0x7e000 ESP8266_NONOS_SDK-3.0.5/bin/blank.bin
```

In case adapter is absent and you received the following message:
```
$ dmesg -W
[  872.296083] usb 1-12: Product: USB2.0-Serial
[  872.311100] ch341 1-12:1.0: ch341-uart converter detected
[  872.325101] usb 1-12: ch341-uart converter now attached to ttyUSB0
```
Run following commands to resolve this issue:
```
$ for f in /usr/lib/udev/rules.d/*brltty*.rules; do
    sudo ln -s /dev/null "/etc/udev/rules.d/$(basename "$f")" 
  done
$ sudo udevadm control --reload-rules
$ sudo systemctl mask brltty.path
```

# Configure

1. Connect ESP-01 with `USB to ESP-01` adapter,  switch on `UART` mode on adapter;
2. Plug-in to USB port;
3. Using any available serial monitor tool (e.g. Arduino IDE or PlatformIO) connect to the board;
4. Connect:
```bash
$ pio device monitor -p /dev/ttyUSB0 -b 115200 --rtscts
```

Check status:
```
AT
OK
```

Check firmware version:
```
AT+GMR
AT version:1.7.5.0(Oct  9 2021 09:26:04)  
SDK version:3.0.5(b29dcd3)  
compile time:Oct 15 2021 18:05:30  
Bin version(Wroom 02):1.7.5
```

Configure WiFi (STA mode):
```
AT+CWMODE=?
+CWMODE:1
AT+CIFSR
+CIFSR:STAIP,"0.0.0.0"  
+CIFSR:STAMAC,"98:f4:ab:ed:31:b2"
AT+CWJAP_DEF?  
No AP
AT+CWLAP
... 
+CWLAP:(3,"ZION",-60,"50:ff:20:62:8d:d8",8,6,0,4,4,7,1)
...
AT+CWJAP_DEF="ZION","<password>"
WIFI CONNECTED  
WIFI GOT IP
OK
AT+CWJAP_DEF?
+CWJAP_DEF:"ZION","50:ff:20:62:8d:d8",8,-68,0
AT+CIFSR
+CIFSR:STAIP,"192.168.1.34"  
+CIFSR:STAMAC,"98:f4:ab:ed:31:b2"
```
