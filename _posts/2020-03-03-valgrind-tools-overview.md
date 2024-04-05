---
title: "Valgrind tools overview"
date: 2020-03-01
categories: [Development,Tools]
tags: [memory,valgrind]
---

# Install

To check for memory leak we will use [valgrind](https://www.valgrind.org/) tools.
First we need to compile and install it:

Install `valgrind`:
``` bash
$ wget https://sourceware.org/pub/valgrind/valgrind-3.16.1.tar.bz2
$ tar -xf valgrind-3.16.1.tar.bz2 && rm valgrind-3.16.1.tar.bz2
$ cd valgrind-3.16.1
$ ./configure
$ make -j$(nproc)
$ sudo make install
```

Install KCachegrind (optional):
```bash
$ sudo apt update
$ sudo apt install kcachegrind
```

Install massif-visualizer (optional):
```bash
$ sudo apt update
$ sudo apt install massif-visualizer
```

# Checking

## Memory checking

[Memcheck](https://www.valgrind.org/docs/manual/mc-manual.html) detects memory-management problems by redefining `malloc/new/free/delete` methods. As a result it can detects:

* Access violation;
* Using uninitialized values in dangerous ways;
* Memory leaks;
* Invalid calls of releasing of heap memory blocks (double frees, mismatched frees);
* Memory overlapping of source and destination memory blocks.

Prepare (+debug; -opmization):
```bash
$ g++ -g -O0 example.cpp -o example.x
```

Using:
```bash
$ valgrind --tool=memcheck --leak-check=yes --show-reachable=yes example.x
```

Options:
```bash
$ valgrind --tool=memcheck --help-dyn-options
```

## Cachegrind

[Cachegrind](https://www.valgrind.org/docs/manual/cg-manual.html) is a cache profiler. It identifies the number of cache misses, memory references and instructions executed for each line of source code, with per-function, per-module and whole-program summaries.

Prepare (-debug; +opmization):
```bash
$ g++ -O2 example.cpp -o example.x
```

Using:
```bash
$ valgrind --tool=cachegrind --branch-sim=yes example.x
```

Above command produces `cachegrind.out.[pid]` file. It has to be opened by:
* `cg_annotate` tool
* [KCachegrind](https://kcachegrind.github.io/html/Home.html) (preferred)

## Callgrind

[Callgrind](https://www.valgrind.org/docs/manual/cl-manual.html) is a profiling tool that records the call history among functions in a program's run as a call-graph.

Prepare (+debug; -opmization):
```bash
$ g++ -g -O0 example.cpp -o example.x
```

Using:
```bash
$ valgrind --tool=callgrind example.x
```

Above command produces `callgrind.out.[pid]` file. It has to be opened by:
* `callgrind_annotate` tool: `callgrind_annotate callgrind.out.[pid]`
* [KCachegrind](https://kcachegrind.github.io/html/Home.html) (preferred)

## Massif checking

[Massif](https://www.valgrind.org/docs/manual/ms-manual.html) is a heap profiler. It performs detailed heap profiling by taking regular snapshots of a program's heap. It produces a graph showing heap usage over time, including information about which parts of the program are responsible for the most memory allocations.

Prepare (+debug; +opmization):
```bash
$ g++ -g -O2 example.cpp -o example.x
```

Using:
```bash
$ valgrind --tool=massif example.x
```

Above command produces `massif.out.[pid]` file. It has to be opened by:
* `ms_print` tool: `ms_print massif.out.[pid]`
* [massif-visualizer](https://github.com/KDE/massif-visualizer)

## DHAT

DHAT is a tool for examining how programs use their heap allocations. It tracks the allocated blocks, and inspects every memory access. It presents information about these blocks such as sizes, lifetimes, numbers of reads and writes, and read and write patterns.

Using this information helps to identify the following characteristics:
* potential process-lifetime leaks: blocks allocated by the point just accumulate;
* excessive turnover: points which chew through a lot of heap;
* excessively transient: points which allocate very short lived blocks;
* useless or underused allocations: blocks which are allocated but not completely filled in
* blocks with inefficient layout - areas never accessed.

Prepare (+debug; +opmization):
```bash
$ g++ -g -O2 example.cpp -o example.x
```

Using:
```bash
$ valgrind --tool=dhat example.x
```

Above command produces `dhat.out.[pid]` file. It has to be opened by: dh_view.html (path provided in commnad output) in browser.


## Helgrind

Helgrind is a tool for detecting synchronisation errors programs that use the POSIX pthreads threading primitives.

Helgrind can detect three classes of errors, which are discussed in detail in the next three sections:
* Misuses of the POSIX pthreads API (mostly, wrong working with mutexes);
* Potential deadlocks;
* Data races.

Prepare (+debug; -opmization):
```bash
$ g++ -g -O0 example.cpp -o example.x
```

Using:
```bash
$ valgrind --tool=helgrind example.x
```

## DRD

DRD is a tool for detecting errors in multithreaded programs.

DRD can detect next classes of errors:
Data races
* Lock contention (one thread holding lock too long thus blocks the progress of other threads);
* Improper use of the POSIX threads API;
* Deadlock;
* False sharing (when threads that runs on different cores access different variables located in the same cache line frequently).

Prepare (+debug; -opmization):
```bash
$ g++ -g -O0 example.cpp -o example.x
```

Using:
```bash
$ valgrind --tool=drd example.x
```



