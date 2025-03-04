---
title: "Manage systemd on Linux system"
date: 2020-02-28
categories: [Engineering,Linux]
tags: [systemd,linux]
---

# Introduction

Architecture of systemd
![architecture-of-systemd]({{site.utl}}/assets/img/posts/systemd1.png)

The systemd startup map
![systemd-startup-map]({{site.utl}}/assets/img/posts/systemd2.png)

Configure grub to see hidden messages and all system startup information:
```shell
$ sudo vim /etc/default/grub
# Delete 'splash' and 'quiet' in
#  GRUB_CMDLINE_LINUX
#  or
#  GRUB_CMDLINE_LINUX_DEFAULT
$ sudo update-grub
```

# View targets

A systemd target represents a current or desired run state.

| systemd<br>targets | Description                                                                                                                                                                                                                                                                                           |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| default.target     | This target is always aliased with a symbolic link to<br>either **multi-user.target** or **graphical.target**.<br>systemd always uses the **default.target** to start the<br>system. The **default.target** should never be aliased<br>to **halt.target**, **poweroff.target**, or **reboot.target**. |
| graphical.target   | **multi-user.target** with a GU                                                                                                                                                                                                                                                                       |
| multi-user.target  | All services running, but command-line interface<br>(CLI) only.                                                                                                                                                                                                                                       |
| rescue.target      | A basic system, including mounting the filesystems<br>with only the most basic services running and a rescue<br>shell on the main console.                                                                                                                                                            |
| emergency.target   | Single-user mode—no services are running;<br>filesystems are not mounted. This is the most basic<br>level of operation with only an emergency shell<br>running on the main console for the user to interact<br>with the system.                                                                       |
| halt.target        | Halts the system without powering it down.                                                                                                                                                                                                                                                            |
| reboot.target      | Reboot.                                                                                                                                                                                                                                                                                               |
| poweroff.target    | Halts the system and turns the power off.                                                                                                                                                                                                                                                             |
List all available targets:
```shell
$ systemctl list-units --type target
```

List dependencies of `graphical.target` target:
```
$ systemctl list-dependencies graphical.target
```

List dependencies of `multi-user.target` target:
```shell
$ systemctl list-dependencies multi-user.target
```

Display default system target:
```shell
$ systemctl get-default
multi-user.target
```

Switch to particular target:
```shell
$ systemctl isolate multi-user.target
```

Change `default.target` from `graphical.target` to `multi-user.target`:
```
# Get default target
$ systemctl get-default
multi-user.target
# Change default target
$ systemctl set-default graphical.target
Removed /etc/systemd/system/default.target.
Created symlink /etc/systemd/system/default.target →
/usr/lib/systemd/system/graphical.target
# Switch to the new default target
$ systemctl isolate default.target
```
# View units

To view all available unit files:
~~~ bash
$ systemctl list-unit-files
~~~
To view only service unit files:
~~~ bash
$ systemctl list-unit-files --type=service
~~~

To list all all running units:
~~~ bash
$ systemctl list-units
~~~
To list failed units:
~~~ bash
systemctl –failed
~~~

# View logs

Display boot and startup message generated by the system during normal operation:
```
$ journalctl
```

To view logs for particular service:
```shell
$ journalctl -u name.service
```

To view error and alert logs for particular service:
```shell
$ journalctl -u name.service -p err..alert
```

To view logs for particular service sing 20 minites ago:
```shell
journalctl -u name.service --since "20 min ago"
```

# Manage services

To get status:
~~~ bash
$ systemctl start name.service
~~~
To start/stop/restart service:
~~~ bash
$ systemctl start name.service
$ systemctl stop name.service
$ systemctl restart name.service
~~~
To reload service unit file changes:
~~~ bash
systemctl reload name.service
~~~
To enable service:
~~~ bash
systemctl enable name.service
~~~
To disable service:
~~~ bash
systemctl disable name.service
~~~

To mask service and prevent it from starting up at all:
~~~ bash
systemctl mask name.service
~~~
To unmask service:
~~~ bash
systemctl unmask name.service
~~~

# Links

[Systemd For Upstart Users](https://wiki.ubuntu.com/SystemdForUpstartUsers) 
