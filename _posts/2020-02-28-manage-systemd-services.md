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

A systemd target represents a current or desired run state:

| Name                | Description                                                                                                                                                                                                                                                                                           |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `default.target`    | This target is always aliased with a symbolic link to<br>either **multi-user.target** or **graphical.target**.<br>systemd always uses the **default.target** to start the<br>system. The **default.target** should never be aliased<br>to **halt.target**, **poweroff.target**, or **reboot.target**. |
| `graphical.target`  | **multi-user.target** with a GU                                                                                                                                                                                                                                                                       |
| `multi-user.target` | All services running, but command-line interface<br>(CLI) only.                                                                                                                                                                                                                                       |
| `rescue.target`     | A basic system, including mounting the filesystems<br>with only the most basic services running and a rescue<br>shell on the main console.                                                                                                                                                            |
| `emergency.target`  | Single-user mode—no services are running;<br>filesystems are not mounted. This is the most basic<br>level of operation with only an emergency shell<br>running on the main console for the user to interact<br>with the system.                                                                       |
| `halt.target`       | Halts the system without powering it down.                                                                                                                                                                                                                                                            |
| `reboot.target`     | Reboot.                                                                                                                                                                                                                                                                                               |
| `poweroff.target`   | Halts the system and turns the power off.                                                                                                                                                                                                                                                             |

| Name                    | Description                                                |
| ----------------------- | ---------------------------------------------------------- |
| `network-online.target` | Target that meets the needs of a fully operational network |

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

Locations:
* `/etc/systemd`
	* `/etc/systemd/system` 
	* `/etc/systemd/user` 
* `/usr/lib/systemd` - unit files
	* `/etc/systemd/network` 
	* `/etc/systemd/system` 
	* `/etc/systemd/user` 

| Unit         | Description                                                                                                                                                                                                                                                                                                             |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `.automount` | The `.automount` units are used to implement on-demand (i.e., plug and play) and mounting of filesystem units in parallel during startup.                                                                                                                                                                               |
| `.device`    | The `.device` unit files define hardware and virtual devices that are exposed to the sysadmin in the `/dev` directory. Not all devices have unit files. Typically, block devices such as hard drives, network devices, and some others have unit files.                                                                 |
| `.mount`     | The `.mount` unit defines a mount point on the Linux filesystem directory structure.                                                                                                                                                                                                                                    |
| `.scope`     | The `.scope` unit defines and manages a set of system processes. This unit is not configured using unit files, rather it is created programmatically. Per the `systemd.scope` man page, "The main purpose of scope units is grouping worker processes of a system service for organization and for managing resources." |
| `.service`   | The `.service` unit files define processes that are managed by systemd. These include services such as `crond` cups (Common Unix Printing System), `iptables`, multiple logical volume management (LVM) services, NetworkManager, and more.                                                                             |
| `.slice`     | The `.slice` unit defines a "slice" which is a conceptual division of system resources that are related to a group of processes. You can think of all system resources as a pie and this subset of resources as a "slice" out of that pie.                                                                              |
| `.socket`    | The `.socket` units define IPC communication sockets, such as network<br>sockets.                                                                                                                                                                                                                                       |
| `.swap`      | The `.swap` units define swap devices or files.                                                                                                                                                                                                                                                                         |
| `.target`    | The `.target` units define groups of unit files that define startup synchronization points, runlevels, and services. Target units define the services and other units that must be active in order to start successfully.                                                                                               |
| `.timer`     | The `.timer` unit defines timers that can initiate program execution at specified times.                                                                                                                                                                                                                                |
To view all available unit files:
~~~ bash
$  systemctl --all -t service
186 loaded units listed.
To show all installed unit files use 'systemctl list-unit-files'.
# or
$ systemctl list-unit-files
~~~

To view unit files with different types:
~~~ bash
$ systemctl list-unit-files -t service
# or
$ systemctl list-unit-files -t timer
# or
$ systemctl list-unit-files -t mount
# 'generated' - unit files generated during startup
# 'static' - unit files for filesystem like /proc and /sys
~~~

To list all all running units:
~~~ bash
$ systemctl list-units
~~~

To list failed units:
~~~ bash
systemctl –failed
~~~

# View timers

* **Absolute timestamp**: A single unambiguous and unique point in time defined in the
format `YYYY-MM-DD HH:MM:SS`. The timestamp format specifies points in time when
events are triggered by timers.
* **Accuracy** is the quality of closeness to the true time; in other words, how close to the
specified calendar time an event is triggered by a timer. The default accuracy for
systemd timers is defined as a one-minute timespan that starts at the defined calendar
time.
* **Calendar** events are one or more specific times specified by a systemd timestamp in
the format `YYYY-MM-DD HH:MM:SS`. It can be a single point in time or a series of
points that are well-defined and for which the exact times can be calculated. In systemd, exact time is specified in the timestamp format `YYYY-MM-DD HH:MM:SS`. When only the YYYY-MM-DD portion is specified, the time defaults to 00:00:00. When only the `HH:MM:SS` portion is specified, the date is the next calendar instance of that time.
* **Timespan** is the amount of time between two events or the duration of something like
an event or the time between two events.

systemd recognizes the following time units:
* usec, us, µs
* msec, ms
* seconds, second, sec, s
* minutes, minute, min, m
* hours, hour, hr, h
* days, day, d
* weeks, week, w
* months, month, M (defined as 30.44 days)
* years, year, y (defined as 365.25 days)
(see systemd.time(7) man page for a complete description)

| Timer              | Monotonic | Definition                                                                                                                                                                                                                                                                                                                                                                              |
| ------------------ | --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| OnActiveSec=       | Yes       | This defines a timer relative to the moment the timer is<br>activated                                                                                                                                                                                                                                                                                                                   |
| OnBootSec=         | Yes       | This defines a timer relative to when the machine boots up                                                                                                                                                                                                                                                                                                                              |
| OnStartupSec=      | Yes       | This defines a timer relative to when the service manager<br>first starts. For system timer units, this is very similar to<br>OnBootSec=, as the system service manager generally starts very early at boot. It's primarily useful when configured in units running in the per-user service manager, as the user service manager generally starts on first login only, not during boot. |
| OnUnitActiveSec=   | Yes       | This defines a timer relative to when the timer that is to be<br>activated was last activated.                                                                                                                                                                                                                                                                                          |
| OnUnitInactiveSec= | Yes       | This defines a timer relative to when the timer that is to be<br>activated was last deactivated.                                                                                                                                                                                                                                                                                        |
| OnCalendar=        | No        | This defines real-time (i.e., wall clock) timers with calendar<br>event expressions. See systemd.time(7) for more information on the syntax of calendar event expressions. Otherwise, the semantics are similar to OnActiveSec= and related settings. This timer is the one most like those used with the cron service.                                                                 |
The list of calendar timer specification examples:

| Calendar event<br>specification | OnCalendarDescription                                                                                                                             |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| DOW YYYY-MM-DD<br>HH:MM:SS      |                                                                                                                                                   |
| `*-*-* 00:15:30`                | Every day of every month of every year at 15 minutes and 30 seconds<br>after midnight                                                             |
| `Weekly`                        | Every Monday at 00:00:00                                                                                                                          |
| `Mon *-*-* 00:00:00`            | Same as weekly                                                                                                                                    |
| `Mon`                           | Same as weekly                                                                                                                                    |
| `Wed 2020-*-*`                  | Every Wednesday in 2020 at 00:00:00                                                                                                               |
| `Mon..Fri 2021-*-*`             | Every weekday in 2021 at 00:00:00                                                                                                                 |
| `2022-6,7,8-1,15 01:15:00`      | The 1st and 15th of June, July, and August of 2022 at 01:15:00am                                                                                  |
| `Mon *-05~03`                   | The next occurrence of a Monday in May of any year which is also the<br>3rd day from the end of the month.                                        |
| `Mon..Fri *-08~04`              | The 4th day preceding the end of August for any years in which it also<br>falls on a weekday.                                                     |
| `*-05~03/2`                     | The 3rd day from the end of the month of May and then again two days<br>later. Repeats every year. Note that this expression uses the Tilde (~).  |
| `*-05-03/2`                     | The third day of the month of may and then every 2nd day for the rest<br>of May. Repeats every year. Note that this expression uses the dash (-). |

To list all all timers:
```shell
$ systemctl list-timers
```

Validating and examining calendar time event specifications in a timer:
```shell
$ systemd-analyze calendar 2030-06-17
  Original form: 2030-06-17
Normalized form: 2030-06-17 00:00:00
    Next elapse: Mon 2030-06-17 00:00:00 EEST
       (in UTC): Sun 2030-06-16 21:00:00 UTC
       From now: 5 years 3 months left
$ systemd-analyze calendar "2027-6,7,8-1,15 01:15:00"
  Original form: 2027-6,7,8-1,15 01:15:00
Normalized form: 2027-06,07,08-01,15 01:15:00
    Next elapse: Tue 2027-06-01 01:15:00 EEST
       (in UTC): Mon 2027-05-31 22:15:00 UTC
       From now: 2 years 2 months left
```

Validating and examining timestamps:
```shell
$  systemd-analyze timestamp "Jun 17 10:08:41"
```
# View logs

Configuration file: `/etc/systemd/journald.conf`.
(see `journald.conf` man page for more details)

Example of journactl configuration:
```text
SystemMaxUse=1G
Storage=persistent
SystemMaxFiles=10
MaxRetentionSec=1month
```
(limit the total amount of storage space by 1Gb, store all journal entries in persistent storage, keep maximum 10 files, delete any journal archive files that are more than a month old)

Most common options:

| Option                                              | Description                                                                                                                                                                                                                                                 |
| --------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `--list-boots`                                      | This displays a list of boots. The information can be used to show journal entries only for a particular boot.                                                                                                                                              |
| `-b [offset\|boot ID]`                              | This specifies which boot to display information for. It includes all journal entries from that boot through shutdown or reboot.                                                                                                                            |
| `--facility=[facility<br>name]`                     | This specifies the facility names as they're known to syslog.                                                                                                                                                                                               |
| `-k, --dmesg`                                       | These display only kernel messages and are equivalent to using the dmesg command.                                                                                                                                                                           |
| `-S, --since<br>[timestamp]`                        | These show all journal entries since (after) the specified time. They can be used with --until to display an arbitrary range of time. Fuzzy times such as "yesterday" and "2 hours ago"—with quotes—are also allowed.                                       |
| `-u [unit name]`                                    | The -u option allows you to select specific units to examine.                                                                                                                                                                                               |
| `-U, --until<br>[timestamp]`                        | These show all journal entries until (prior to) the specified time. They can be used with --since to display an arbitrary range of time. Fuzzy times such as "yesterday" and "2 hours ago"—with quotes—are also allowed.                                    |
| `-f, --follow`                                      | This journalctl option is similar to using the tail -f command. It<br>shows the most recent entries in the journal that match whatever other options have been specified and also displays new entries as they occur.                                       |
| `-e, --pager-end`                                   | The -e option displays the end of the data stream instead of the beginning.                                                                                                                                                                                 |
| `--file [journal filename]`                         | This names a specific journal file in `/var/log/journal/<journal<br>subdirectory>`.                                                                                                                                                                         |
| `-r, --reverse`                                     | This option reverses the order of the journal entries in the pager<br>so that the newest are at the top rather than the bottom.                                                                                                                             |
| `-n, --lines=[X]`                                   | This shows the most recent X number of lines from the journal.                                                                                                                                                                                              |
| `--utc`                                             | This displays times in UTC rather than local time.                                                                                                                                                                                                          |
| `-g, --grep=[REGEX]`                                | Enables to search for specific patterns in the journal data stream. This is just like piping a text data stream through the grep command. This option uses Perl-compatible regular expressions.                                                             |
| `--disk-usage`                                      | This option displays the amount of disk storage used by the<br>current and archived journals. It might not be as much as you<br>think.                                                                                                                      |
| `--flush`                                           | Journal data stored in the virtual filesystem `/run/log/journal`,<br>which is volatile storage, is written to `/var/log/journal` which is persistent storage. This option ensures that all data is flushed<br>to `/run/log/journal` at the time it returns. |
| `--sync`                                            | This writes all unwritten journal entries (still in RAM but not in<br>`/run/log/journal`) to the persistent filesystem. All journal entries known to the journaling system at the time the command is entered are moved to persistent storage.              |
| `--vacuum-size= --<br>vacuum-time= --vacuum-files=` | These can be used singly or in combination to remove the oldest archived journal files until the specified condition is met. These options only consider archived files, and not active files, so the result might not be exactly what was specified.       |
Displays boot and startup logs generated by the system during normal operation:
```
$ journalctl
```

Display logs for specific identifier (program or script name):
```shell
$ journalctl -t <identifier>
```

Displays most recent logs for the specified syslog indentifier:
```shell
$ journalctl -b -t <indentifier>
```

Displays logs after filtering by given clause:
```shell
$ journalctl -g "hello" 
```

Displays logs by specifying a time range: 
```shell
$ journalctl -g "hello" \
--since="<time specification>" \
--until="<time specification>"
# e.g. "<time specification>" = "2020-05-10 10:54:35"
# e.g. "<time specification>" = "today"
```

Displays logs for particular service:
```shell
$ journalctl -u name.service
```

Displays logs for particular service of current boot:
```shell
$ journalctl -u name.service -b
```

Displays error and alert logs for particular service:
```shell
$ journalctl -u name.service -p err..alert
```

Displays logs for particular service since 20 minutes ago:
```shell
journalctl -u name.service --since "20 min ago"
```

Displays logs for particular service between two  arbitrary times:
```shell
journalctl -u name.service -S "2020-07-24 15:36:00" -U "2020-07-25 16:00:00"
```

Displays logs since the latest system boot:
```shell
$ journalctl -b
or
$ journalctl -b -o short-monotonic
```

Displays `dmesg` logs:
```shell
$ journalctl --dmesg
# or
$ journalctl --dmesg -o short-monotonic
# -o <format>: set output format, available formats:
#   short, short-full, short-iso, short-iso-precise, short-monotonic,
#   short-precise, short-unix, cat, verbose
# -x: shows explanatory messages 
```

List all available boots during the time preiod:
```shell
$ journalctl --list-boots
```

Displays journal for the boost instance with the offset of `-2` (the second previous boost from the current one):
```shell
$ journalctl -b -2
# or
$ journalctl -b <UUID>
```

Display logs of particular facility
```shell
# List all available facility options
$ journalctl --facility=help
# Example
$ journalctl --facility=mail
```

Add journal entry manually:
```shell
$ echo "Hello world" | systemd-cat -p info -t myprog
$ journalctl -n 10
...
Jul 27 09:07:53 testvm1.both.org unknown[976501]: Hello world
```

Rotate logs manually:
```shell
$ journalctl --flush
$ journalctl --rotate
```

Purge old archive files with logs older than 1 month:
```shell
journalctl --vacuum-time=1month
```

Deletes old archive files except the four most recent ones:
```shell
$ journalctl --vacuum-files=4
```

Deletes archive files until 200MB or less of archived files are left:
```shell
$ journalctl --vacuum-size=200M
```
# Manage services

List:
```shell
$ systemctl -t service
# or
$ systemctl -t mount
# (...)
```

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

# Manage performance

Displays general information (how much time each stage takes):
```
$ systemd-analyze
Startup finished in 6.350s (firmware) + 7.572s (loader) + 2.598s (kernel) + 2min 33.334s (userspace) = 2min 49.855s 
graphical.target reached after 2min 33.315s in userspace.
```

Displays which systemd units take the most time to initialize:
```shell
$ systemd-analyze blame
# or to get specific service blame
$ systemd-analyze blame | grep "some.service"
```

Displays the time-critical chain of events:
```shell
$ systemd-analyze critical-chain
...
# @<value>: the absolute number of seconds since startup
# +<value>: the amount of time it takes for the unit to start
# or to analyze service critical chain
$ systemd-analyze critical-chain some.service
```

Displays the system state:
```shell
$ systemd-analyze dump
```

Generates a vector graphics file with events that take place during boot and startup:
```shell
$ systemd-analyze plot > /tmp/bootup.svg
```

Checks system conditions (kernel version between 4.0 and 5.1, host is running on AC power, system architecture is not ARM and `/etc/os-release` dir exists):
```shell
$ systemd-analyze condition 'ConditionKernelVersion = ! <4.0' \
'ConditionKernelVersion = >=5.1' \
'ConditionACPower=|false' \
'ConditionArchitecture=|!arm' \
'AssertPathExists=/etc/os-release' ; \
echo $?
test.service: AssertPathExists=/etc/os-release succeeded.
Asserts succeeded.
test.service: ConditionArchitecture=|!arm succeeded.
test.service: ConditionACPower=|false failed.
test.service: ConditionKernelVersion=>=5.1 succeeded.
test.service: ConditionKernelVersion=!<4.0 succeeded.
Conditions succeeded.
0
```
(see systemd.unit(5) man page for details)

Checks unit file syntax:
```shell
$ systemd-analyze verify /etc/systemd/system/backup.service
```

Checks security level of specified service:
```shell
$ systemd-analyze security display-manager
```

Lists all slices:
```shell
$ systemctl -t slice --all
```

Displays cgroups hierarchy:
```shell
$ systemd-cgls
```

Links:
* [A Linux sysadmin's introduction to cgroups](https://www.redhat.com/sysadmin/cgroups-part-one)
* [How to manage cgroups with CPUShares](https://www.redhat.com/sysadmin/cgroups-part-two)
* [Managing cgroups the hard way—manually](https://www.redhat.com/sysadmin/cgroups-part-three)
* [Managing cgroups with systemd](https://www.redhat.com/sysadmin/cgroups-part-four)
# Examples

## Start after network is ready:

```
...
After=network-online.target
Wants=network-online.target
...
```

## Timer unit file

File: `/etc/systemd/system/monitor.service`:
```text
[Unit]
Description=Logs system statistics to the systemd journal
Wants=monitor.timer

[Service]
Type=oneshot
ExecStart=/usr/bin/free

[Install]
WantedBy=multi-user.target
```

File: ` /etc/systemd/system/monitor.timer`:
```shell
[Unit]
Description=Logs some system statistics to the systemd journal
Requires=monitor.service

[Timer]
Unit=monitor.service
OnCalendar=*-*-* *:*:00

[Install]
WantedBy=timers.target
```

```shell
$ systemctl daemon-reload
$ systemctl enable monitor.timer
```
## Service unit file

Create the script to emulate program:
```shell
$ sudo tee > /dev/null /usr/local/bin.hello.sh <<EOF
#!/usr/bin/bash
# Simple program to use for testing startup configurations
# with systemd.
# By David Both
# Licensed under GPL V2
#
echo "######### Hello World! ########"
EOF
```
Create service unit file:
```shell
$ sudo tee > /dev/null /etc/systemd/system/hello.service <<EOF
# Simple service unit file to use for testing
# startup configurations with systemd.
# By David Both
# Licensed under GPL V2
#
[Unit]
Description=My hello shell script

[Service]
Type=oneshot
ExecStart=/usr/local/bin/hello.sh

[Install]
WantedBy=multi-user.target
EOF
$ systemctl daemon-reload
$ systemctl status hello.service
```

## Mount CIFS remote share

File: `/etc/systemd/system/media-home.mount`
```shell
[Unit]
Description=Mount 'HOME' remote share
Requires=network-online.target
After=network-online.service
Conflicts=umount.target
Before=umount.target

[Mount]
What=//192.168.1.1/HOME
Where=/media/home
Type=cifs
Options=uid=100000,gid=100000,credentials=/root/.smbcredentials
TimeoutSec=5

[Install]
WantedBy=multi-user.target
```

File: `/etc/systemd/system/media-home.automount`:
```shell
[Unit]
Description=Automount 'HOME' remote share
Requires=network-online.target

[Automount]
Where=/media/home
TimeoutIdleSec=30

[Install]
WantedBy=remote-fs.target
```

## Backup service

File: `/etc/systemd/system/SERVICE-bkp.service`:
```text
[Unit]
Description=SERVICE backup
After=local-fs.target

[Service]
ExecStartPre=/usr/bin/systemctl stop SERVICE
ExecStart=/bin/bash -c 'tar -czf /opt/bkp/SERVICE_$$(date +%%Y-%%m-%%d_%%H-%%M-%%S).tar.gz /opt/SERVICE/'
ExecStartPost=/usr/bin/systemctl start SERVICE
Type=oneshot
```

File: `/etc/systemd/system/SERVICE-bkp.timer`:
```text
[Unit]
Description=SERVICE backup

[Timer]
OnCalendar=*-*-* 05:00:00

[Install]
WantedBy=timers.target
```

# Links

[Systemd For Upstart Users](https://wiki.ubuntu.com/SystemdForUpstartUsers) 
