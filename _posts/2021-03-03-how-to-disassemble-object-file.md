---
title: "How to disassemble object file"
date: 2021-03-03
categories: [Development,Debug]
tags: [debug]
---

First of all we need object file without any opmization and linking with standard library:
```bash
$ g++ -c -g -O0 file.cpp
```
then we use `objdump` and disassemble object file with helpful annotation:
```bash
$ objdump -gdSC -M intel file.o > file.d
```
Also we can get code by lines with particular address of instruction in assemble code:
```bash
$ readelf --debug-dump=decodedline file.o > file.lines
```