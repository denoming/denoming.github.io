---
title: "Install WiFi adapter"
date: 2026-11-05
categories: [Engineering,Linux]
tags: [wifi]
---

Switch module from "USB storage" mode to adapter mode:
```
$ lsusb
...
Bus 001 Device 008: ID 1111:1111 Pandora International Ltd. 88M80
$ sudo usb_modeswitch -v 1111 -p 1111 \
  -M "555342438765432100000000000010fd0000000000000000000000000000f3" \
  -2 "555342438765432100000000000010fd0000000000000000000000000000f2"
# Unplag and plug
# Apply operation again if it did not work

The ID must change from 1111:1111 to something else, for example:
$ lsusb
...
Bus 001 Device 010: ID a69c:8d80 aicsemi AIC Wlan
```

Install driver:
```
$ git clone https://github.com/shenmintao/aic8800d80.git
$ cd aic8800d80
$ sudo ./install.sh
```


Connect to WiFi network:
```
$ nmcli device status
...
wlan0          wifi      disconnected
p2p-dev-wlan0  wifi-p2p  disconnected
$ nmcli dev wifi
...
58:8B:F3:66:84:48  XXXX                             Infra  2     130 Mbit/s  95
$ sudo nmcli dev wifi connect "XXXX" password "<password>"
```

Disconnect from WiFi network:
```
$ sudo nmcli device disconnect wlan0
```

Create WiFi profile instead of connecting directly:
```
sudo nmcli connection add type wifi ifname wlan0 con-name "YYYY" ssid "XXXX" -- wifi-sec.key-mgmt wpa-psk
sudo nmcli connection modify "YYYY" wifi-sec.psk "<password>"
sudo nmcli connection up "YYYY"
```

Reduce the metric for wired connection and increase metric for WiFi network:
```
$ ip route
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.46 metric 200
$ sudo nmcli connection modify "Wired connection 1" ipv4.route-metric 200
$ sudo nmcli connection modify "YYYY" ipv4.route-metric 100
$ sudo nmcli connection up "YYYY"
$ sudo nmcli connection up "Wired connection 1"
$ sudo nmcli connection modify "Wired connection 1" ipv4.never-default yes
$ sudo nmcli connection up "Wired connection 1"
```


Checking the network route:
```
$ sudo apt install -y traceroute
$ ip route get 8.8.8.8
8.8.8.8 via 10.75.104.188 dev wlan0 src 10.75.104.49 uid 1000 
    cache 
$ traceroute 8.8.8.8
```