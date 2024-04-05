---
title: "Useful Linux commands"
date: 2020-04-07
categories: [Engineering, Linux]
tags: [linux]
---

# Useful Linux commands

## General

### What command is?

`which` - shows the first command found in PATH variable
```bash
$ which awk
/usr/bin/awk
$ which -a awk
/usr/bin/awk
/bin/awk
/usr/bin/X11/awk
```

`whereis` - shows more details information about command (can search man pages)
```bash
$ whereis awk
awk: /bin/awk /usr/bin/awk /usr/lib64/awk /usr/bin/X11/awk /usr/share/awk 
/usr/share/man/man1/awk.1.gz /usr/share/man/man1p/awk.1p.gz
```

### How to find file?

`find` - search for file/directory
```bash
$ find /tmp -user ian # search by user owner
/tmp/kde-ian
...
$ find -L research -maxdepth 2 -type d ! -group ian # search directory that doesn't have 'ian' as group owner
research/rh62/involution
research/rh62/programs
research/lost+found
```

`locate` - search file in database that updated by cron
`updatedb` - update file database
```bash
$ locate /bin/ls
/bin/ls
/bin/lsmod
```

## Process

List of signals:
```bash
$ kill -l
 1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
 6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
...
```

How to send signal by keyboard:
```
CTRL+C - SIGINT (terminate process)
CTRL+Z - SIGTSTP (suspend all operations)
CTRL+\ - SIGQUIT (quit immediately)
```

`pgrep` - find process by name 
```bash
$ pgrep -l skype
19126 skypeforlinux
```

`pkill` - terminate process by name
```bash
$ pkill skype
```

