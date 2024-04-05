---
title: "How to use addr2line tool"
date: 2021-03-03
categories: [Development,Debug]
tags: [addr2line,debug]
---

The `addr2line` tool is used to get the line in code by the address which is obtained in several ways. One of the way to obtain address is get it from `dmesg` log when segmentation fault has happened.
For our example we will build and run simple program with invalid pointer dereference. Pay attention we are going to build our program with debug information:
```bash
$ g++ -g main.cpp -o main.x
$ ./main.x
Segmentation fault (core dumped)
```

Get two last logs from `dmesg`:
```bash
$ dmesg | tail -2
[ 4930.533633] main.x[9628]: segfault at dadabaef ip 000055a6f136c1f8 sp 00007ffdc0461270 error 4 in main.x[55a6f136c000+1000]
[ 4930.533647] Code: 48 89 e5 48 83 ec 10 b8 ef ba da da 48 89 45 f8 48 8d 35 20 0e 00 00 48 8d 3d 54 2e 00 00 e8 af fe ff ff 48 89 c2 48 8b 45 f8 <8b> 00 89 c6 48 89 d7 e8 cc fe ff ff 48 89 ...
```

These are some hints in dmesg output:
* `main.x` is the executable name
* `9628` is the process ID
* `dadabaef` is the faulty address in hexadecimal
* `ip` - the instruction pointer
* `sp` - the stack pointer
* `error 4` is an error code
* the string at the end is the name of the virtual memory area - "in {file name}[{address} + {offset}]"

To find out the address we have to follow this formula:
	IP - VMA(address) + VMA(offset) => (needs clarification)
We put values into formula and receive:
	55a6f136c1f8 - 55a6f136c000 + 1000 = 11F8

Using `addr2line` and address `11F8` we can obtain the row of code:
```bash
$ addr2line -e main.x 11f8
...main.cpp:8
$ sed '8!d' main.cpp
   std::cout << "Value: " << *p << std::endl;
```

The error code is a combination of several error bits defined in [fault.c](https://elixir.bootlin.com/linux/v4.11/source/arch/x86/mm/fault.c#L32):
```c
/*
 * Page fault error code bits:
 *
 *   bit 0 ==	0: no page found	   1: protection fault
 *   bit 1 ==	0: read access		   1: write access
 *   bit 2 ==	0: kernel-mode access  1: user-mode access
 *   bit 3 ==	                       1: use of reserved bit detected
 *   bit 4 ==				           1: fault was an instruction fetch
 *   bit 5 ==				           1: protection keys block access
 */
enum x86_pf_error_code {

	PF_PROT		=		1 << 0,
	PF_WRITE	=		1 << 1,
	PF_USER		=		1 << 2,
	PF_RSVD		=		1 << 3,
	PF_INSTR	=		1 << 4,
	PF_PK		=		1 << 5,
};
```

Since we are executing a user-mode program, `PF_USER` is set and the error code is at least 4. Moreover, if the error is 4, we have read access. If the invalid memory access is a write, then `PF_WRITE` is set.
