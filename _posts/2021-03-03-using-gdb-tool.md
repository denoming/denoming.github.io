---
title: Using GDB tool
date: 2021-03-03
categories:
  - Development
  - Debug
tags:
  - gdb
  - debug
---
# Install

```shell
$ sudo apt install gdb
$ gdb --version
GNU gdb (Ubuntu 15.0.50.20240403-0ubuntu1) 15.0.50.20240403-git
```

```
-g tells to include:
 * symbol names
 * type info for symbols
 * files name has line number where the symbols come from
```
# Using
## Compile

```bash
$ g++ -g -O2 app.cpp -o app
c v ```
## Start

```shell
$ gdb --args app [--arg1 --arg1]
...
(gdb) show args
Argument list to give program being debugged when it is started is
  " --arg1 --arg1".
(gdb) break main
Breakpoint 1 at ...
(gdb) run
```
## Commands

### General

```shell
# List of functions
(gdb) info functions
# List of source code (or current frame)
(gdb) list
or
(gdb) list <line-from>[,<line-to>]
or
(gdb) list <function-name>
```

```shell
# Show help
(gdb) help <command>
```

```shell
# Show disassemble code
(gdb) disassemble main
```

```shell
# Save GDB commands to a gdb.txt file
(gdb) set logging on
...
(gdb) set logging off
```

```shell
# Save GDB commands to a <file> file
(gdb) set logging file <file>
(gdb) set logging on
...
(gdb) set logging off
```

```shell
# Attach to process
(gdb) sudo gdb
(gdb) shell pgrep <program-name>
... <PID>
(gdb) attach <PID>
or
$ kll -s SIGABRT <PID>
or
by "abort()" call inside the program
```

```shell
# Generate a coredump in a particular time
(gdb) sudo gdb
(gdb) shell pgrep <name>
... <PID>
(gdb) attach <PID>
(gdb) generate-core-file <optional-filename>
(gdb) detach
```

### Execute

```
# Put a brakpoint into 'main' and execute binary
(gdb) start
# Execute binary (or restart from start)
(gdb) run [arg1 ...]
# Execute next line (without going inside if the next line is a function)
(gdb) next
# Execute next line (with going inside if the next line is a function)
(gdb) step
# Continue execution till next breakpoint or exit
(gdb) continue
# Continue until function end
(gdb) finish
```
### Breakpoints

```shell
# Put a breakpoint
(gdb) break <functiona-name>
or
(gdb) break [<filename>:]<line-number>
```

```shell
# List of breakpoints
(gdb) info breakpoints
# List all global and static variables variables
(gdb) info variables
# List variable of current stack frame
(gdb) info locals
# List argument values of current stack frame
(gdb) info args
# List register values
(gdb) info register values
# Get information about the frame
(gdb) info frame <number>

```

```shell
# Delete a breakpoint
(gdb) detele <list-index-of-breakpoint>
```

```shell
# Add conditional breakpoint
(gdb) condition <breakpoint-index> <variable>==<value>
```

### Commands

```shell
# Create a command at line <line> that print <variable> value
(gdb) breakpoint <line>
(gdb) command 1
> print <variable>
> end
(gdb) run
```

### Watch points

Watch points are set on variables. When those variables are read/write, the watch point is triggered and program execution stops. To set a watch point of a non-global variable, first a breakpoints must be set, that will stop execution of a program when the variable is in scope.

```shell
# Set watchpoint
(gdb) watch <variable>
# Set a read watchpoint
(gdb) rwatch <variable>
# Set a read/write watchpoint
(gdb) awatch <variable>
# Enable/disable a watchpoint
(gdb) enable/disable <watchpoint-index>
```

### Coredump

```shell
# Get backtrace
(gdb) backtrace
or
(gdb) where
```

```shell
# Select specific stack frame
(gdb) frame <index-of-frame>
```

```shell
# Show the type of a variable
(gdb) ptype <variable-name>
type = int
(gdb) ptype &<variable-name>
type = int *
(gdb) ptype <function-name>
type = int ()
```

```shell
# Show the value of variable
(gdb) print <variable-or-expression>
or
(gdb) print &<variable>
or
(gdb) print sizeof(<variable>)
or
(gdb) print sizeof(&<variable>)
or
(gdb) print[/format] <variable>
```

Examples:
```shell
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

```shell
# Change the value of variable
(gdb) set <variable>="<value>"
```

```shell
# Examine memory starting from a particular address
(gdb) examine &<variable>
or
(gdb) examine [/format] &<variable> (/4xb - examine 4 values formatted as hex, one byte at a time)
(e.g. "x/s arg[0]")
```

### UI

* display console interface: `Ctrl+X; Ctrl+A`
* clear/redraw display: `Ctrl+L`

```shell
$ gdbtui
or
$ gdb -tui
or
$ gdb
(gdb) <CTRL+X and A>
```
