---
title: Using trace tools
date: 2025-03-14
categories:
  - My
tags:
  - strace
  - ltrace
---

# strace

`strace` - system call tracer. Each line of `strace` output is a call with a name of method, arguments and output of a call.

Goals:
* learn which system calls a program make
* find those system calls that fail together with error code
* find which files a program open
* find what syscalls a running program is making

Run strace (+child process monitoring):
```shell
$ strace -f <program> [<arg1>...]
or
$ strace -f -s 500 <program>
```

Run strace and save output to the given file:
```shell
$ strace -ff -o <output-file> <program>
```

Run strace with getting time psent per system call:
```shell
$ strace -Ttt <program>
```

Get running statistic:
```shell
# statistic+output
$ strace -c <command>
# statistic+traces
$ strace -C <command>
```

Attach to running process:
```shell
$ sudo strace -p <PID>
```

Tracing only specific system calls:
```shell
# Trace only open and close system calls
$ strace -e trace="open,close" <program>
# or to negate
$ strace -e trace="!open,close" <program>

# -e trace=ipc      # trace communication between processes
# -e trace=memory   # trace memory system calls
# -e trace=network  # trace network system calls
# -e trace=process  # trace process calls (e.g. fork, exec)
# -e trace=signal   # trace process signal handling
# -e trace=file     # trace system calls that includes filenames
```

# ltrace

`ltrace` - library call tracer. Each line of `ltrace` output is a call with a name of method, arguments and output of a call that is made against particular library.

```shell
$ ltrace <program> [<arg1>...]
```

Trace only specific calls:
```shell
$ ltrace -e <name> <program>
```
  
  Attach to running process:
```shell
$ sudo ltrace -p <PID>
```