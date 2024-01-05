---
title: "How to monitor HTTP traffic"
date: 2022-06-19
categories: [Engineering,Linux]
tags: [http,tcpdump]
---

* To monitor HTTP traffic including request and response headers and message body:
```sh
tcpdump -A -s 0 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
```
Note: Change "tcp port 80" to "tcp dst port 80" to include only requests.

* To monitor HTTP traffic including request and response headers and message body from a particular source:
```sh
tcpdump -A -s 0 'src example.com and tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
```
Note: Change "tcp port 80" to "tcp dst port 80" to include only requests.

* To monitor HTTP traffic including request and response headers and message body from local host to local host:
```sh
tcpdump -A -s 0 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)' -i lo
```
Note: Change "tcp port 80" to "tcp dst port 80" to include only requests.

* Capture TCP packets from local host to local host:
```sh
tcpdump -i lo
```