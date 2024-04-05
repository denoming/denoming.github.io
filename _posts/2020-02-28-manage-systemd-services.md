---
title: "Manage systemd on Linux system"
date: 2020-02-28
categories: [Engineering,Linux]
tags: [systemd,linux]
---

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

# Show service logs

To view logs for particular service:
~~~ bash
$ journalctl -u name.service
~~~
To view error and alert logs for particular service:
~~~ bash
$ journalctl -u name.service -p err..alert
~~~
To view logs for particular service sing 20 munites ago:
~~~ bash
journalctl -u name.service --since "20 min ago"
~~~

# Links

[Systemd For Upstart Users](https://wiki.ubuntu.com/SystemdForUpstartUsers) 
