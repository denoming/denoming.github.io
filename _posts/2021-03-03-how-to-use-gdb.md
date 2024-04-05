---
title: "How to use GDB tool"
date: 2021-03-03
categories: [Development,Debug]
tags: [gdb,debug]
---

## Compile

```bash
$ g++ -g -O2 app.cpp -o app.x
```

## Start

```bash
$ gdb --args app.x --arg1 --arg1
...
(gdb) show args
Argument list to give program being debugged when it is started is
  " --arg1 --arg1".
(gdb) b main
Breakpoint 1 at ...
(gdb) run
```

## Useful

### Useful hot-keys

* display console interface: `Ctrl+X; Ctrl+A`
* clear display: `Ctrl+L`

### Useful commands

* reach main and stop: `start`
* run until break: `run` (or short `r`)
* step out: `next` (or short: `n`)
* print variable" `print <name>` (or short: `p <name>`)
Examples:
```
(gdb) p i # print variable name
$2 = 0
(gdb) p arr # print array
$6 = {64, 34, 25, 12, 22, 11, 90}
(gdb) p *arr@2 # print given number of elements from array
$10 = {64, 34}
(gdb) p arr[3] # print particular element from array (zero-based)
$11 = 12
(gdb) p (a[3] + 1) # evaluate value
```
* assign own variable: `set $<name> = <value>`
Example:
```
(gdb) set $foo = 4
(gdb) p $foo
$3 = 4
```
* break by name: `break <name> (if <condition>)`  (or short `b <name> (if <condition>)`)
Example:
```
(gdb) b bubbleSort
(gdb) r
```
Or
```
(gdb) b bubbleSort if i == 5
(gdb) r
```

* break by line: `break <number> (if <condition>)` (or short `b <number> (if <condition>)`)
Example:
```
(gdb) b 19
```
* show break points: `info breakpoints`
Example:
```
(gdb) info breakpoints
Num     Type           Disp Enb Address            What
2       breakpoint     keep y   0x000055555555521a in bubbleSort(int*, int) at sort.cpp:14
....

```
* delete break-point: `delete <num>` (or short `d <num>`)
The `<num>` is the name of break-point or watch-point from info table (see `info` command).
* assign command to break-point: `command <num>`
Example:
```
(gdb) set pagination off
(gdb) b 21 # set breakpoint
(gdb) c    # continue to break-point
(gdb) command 3
> p *arr@7
> end
(gdb) c    # continue to break-point
$2 = {25, 12, 22, 11, 34, 64, 90}
(gdb) c    # continue to break-point
$3 = {12, 22, 11, 25, 34, 64, 90}
```
* continue until interruption: `continue` (or short `c`)
