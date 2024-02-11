---
title: "Systemd cheat-sheet"
date: 2020-09-05
categories: [Engineering,Linux]
tags: [rdp]
---

# Introduction

**Units locations:**
* `/lib/systemd/system` - default system location
* `/etc/systemd/system` - preferable location
* `/run/systemd/system` - runtime location (lost on reboot)

To override default system unit file put a replacement in `/etc/systemd/system` dir.

To override only specific directives provide unit file snippets within specific subdirectory.
Create a directory with `.d` prefix (e.g. `example.service.d` for `example.service`).
Within this directory a file ending with `.conf` should be used to override or extend the attributes
of the system's unit file.

**Types of units**:
* `*.service` - describes a service of application 
* `*.socket` - describes a network or IPC socket or FIFO buffer (socket base activation)
* `*.device` - describes a device dependency (udev or sysfs management)
* `*.mount` - defines a mountpoint on the system to be managed by systemd (`<mount-path-with-dashes>.mount`)
* `*.automount` - configures a mountpoint to mount automatically
* `*.swap` - describes swap space on the system (must reflect the device or file path of the space)
* `*.target` - provides synchronization points for other units when booting up or changing states
* `*.path` - defines a path that can be used for path-based activation
* `*.timer` - defines a timer for delayed or scheduled activation
* `*.snapshot` - allows reconstruction of the current state of the system after making changes
* `*.slice` - provides resources limitation to any associated processes (associated with Linux Control Group nodes)
* `*.scope` - used to manage sets of system processes that are created externally

# Service unit

Format:
```text
[Section]
Directive1=value
Directive2=value
. . .
```

Sections:
* `[Unit]` - defines metadata and configuring the relationship towards other units
* `[Install]` (optional) - defines the behaviour when it is enabled ot disabled

## `[Unit]` section

Directives:
* `Description=` - describes the name and basic functionality of the unit
* `Documentation=` - provides a location for a list of URIs for documentation
* `Requires=` - lists any units upon which this unit essentially depends (must successfully activate)
* `Wants=` - less strict dependencies (systemd attempts to start dependency and finally start current unit)
* `BindsTo=` -  causes the current unit to stop when the associated unit terminates
* `Before=` - causes to start before listed units (do not imply dependency relationship)
* `After=` - causes to start after listed units (do not imply dependency relationship)
* `Conflicts=` - lists units that cannot be run at the same time
* `Condition...=` - allows to test certain conditions prior to start the unit
* `Assert...=` - causes a failure in negative result

## `[Install]` section

Directives:
* `WantedBy=` - specifies a dependency relationship (similar to `Wants=` but use different approach)
* `RequiredBy=` - specifies a required dependency (use the same approach as `WantedBy=`)
* `Alias=` - specifies the alias of the unit (brings multiple providers of the same service)
* `Also=` - allows units to be enabled or disabled as a set
* `DefaultInstance=` - specifies fallback value for the name

## `[Service]` section

Main directive is `Type=`. It specifies the type based on the process behavior. Has the following possible values:
* `simple` - the process is specified in the start line
* `forking` - the parent service process forks a child process, exiting the parent process almost immediately, tells `systemd` that the process is running despite main process exited
* `oneshot` - the process will be short-lived, so `systemd` should wait for the process to exit before continuing
* `dbus` - indicates that the service will take a name on the D-Bus bus, then `systemd` will continue
* `notify` - indicates that the service will issue a notification, only then `systemd` continues
* `idle` - indicates that the service will not be run until all jobs are dispatched

Additional directives when using certain service types:
* `RemainAfterExit=` - commonly used with `oneshot` type, indicates that the service should remain active even after the process exit
* `PIDFile=` - commonly used with `forking`, specifies the path where process ID should be placed
* `BusName=` - specifies the D-Bus bus name the service will attempt to acquire
* `NotifyAccess=` - specifies access to the socket that should be used for notification when `notify` type is used (can be `none`, `main` or `all`)

Directives to manage the service:
* `ExecStart=` - the full path and the arguments of the command to be executed to start the process (only one except `oneshot` service type)
* `ExecStartPre=` - specifies additional command to run before executing the main process (can be used multiple times)
* `ExecStartPost=` - specifies additional command to run after the main process is started
* `ExecReload=` - specifies the command that is necessary to reload the configuration of the service
* `ExecStop=` - specifies the command needed to stop the service (if not se the process will be killed immediately)
* `ExecStopPost=` - specifies the command to execute following the stop command
* `RestartSec=` - specifies the amount of time to wait before attempting to restart the service when automatically restarting the service is enabled
* `Restart=` - specifies the circumstances under which `systemd` will attempt to restart the service
  * `always`
  * `on-success`
  * `on-failure`
  * `on-abnormal`
  * `on-abort`
  * `on-watchdog`
* `TimeoutSec=` - configures the amount of time to wait before marking the service as failed

## `[Socket]` section

To specify actual socket:
* `ListenStream=` - defines an address for service that use TCP 
* `ListenDatagram=` - defines an address for service that use UDP
* `ListenSequentialPacket=` - defines an address for sequential and reliable communication (commonly for unit sockets) 
* `ListenFIFO=` - specifies a FIFO buffer instead of a socket

Additional directives to control other characteristics:
* `Accept=` - specifies whether an additional instance for each connection should be created or not
* `SocketUser=` - specifies the owner of the socket
* `SocketGroup=` - specifies the group owner of the socket
* `SocketMode=` - specifies the permissions on the created entity (for unit sockets or FIFO buffers)
* `Service=` - alternative name of the service

## `[Mount]` section

Mount units allow for mount point management from within systemd.
Mount points are named after the directory that they control, with a translation algorithm applied.

* `What=` - the absolute path to the resource that needs to be mounted.
* `Where=` - the absolute path of the mount point where the resource should be mounted. This should be the same as the unit file name, except using conventional filesystem notation.
* `Type=` - the filesystem type of the mount.
* `Options=` - any mount options that need to be applied. This is a comma-separated list.
* `SloppyOptions=` - a boolean that determines whether the mount will fail if there is an unrecognized mount option.
* `DirectoryMode=` - if parent directories need to be created for the mount point, this determines the permission mode of these directories.
* `TimeoutSec=` - configures the amount of time the system will wait until the mount operation is marked as failed.

## `[Automount]` section

This unit allows an associated `.mount` unit to be automatically mounted at boot.

* `Where=` - the absolute path of the automount point on the filesystem
* `DirectoryMode=` - if the automount point or any parent directories need to be created, this will determine the permissions settings of those path components

## `[Swap]` section

Swap units are used to configure swap space on the system. 

`What=` - the absolute path to the location of the swap space, whether this is a file or a device.
`Priority=` - this takes an integer that indicates the priority of the swap being configured.
`Options=` - any options that are typically set in the /etc/fstab file can be set with this directive instead. A comma-separated list is used.
`TimeoutSec=` - the amount of time that systemd waits for the swap to be activated before marking the operation as a failure.

## `[Path]` section

A path unit defines a filesystem path that `systmed` can monitor for changes.
Another unit must exist that will be activated when certain activity is detected at the path location.

`PathExists=` - this directive is used to check whether the path in question exists. If it does, the associated unit is activated.
`PathExistsGlob=` - this is the same as the above, but supports file glob expressions for determining path existence.
`PathChanged=` - this watches the path location for changes. The associated unit is activated if a change is detected when the watched file is closed.
`PathModified=` - this watches for changes like the above directive, but it activates on file writes as well as when the file is closed.
`DirectoryNotEmpty=` - this directive allows systemd to activate the associated unit when the directory is no longer empty.
`Unit=` - this specifies the unit to activate when the path conditions specified above are met. If this is omitted, systemd will look for a .service file that shares the same base unit name as this unit.
`MakeDirectory=` - this determines if systemd will create the directory structure of the path in question prior to watching.
`DirectoryMode=` - if the above is enabled, this will set the permission mode of any path components that must be created.

## `[Timer]` section

Timer units are used to schedule tasks to operate at a specific time or after a certain delay.

`OnActiveSec=` - this directive allows the associated unit to be activated relative to the .timer unit’s activation.
`OnBootSec=` - this directive is used to specify the amount of time after the system is booted when the associated unit should be activated.
`OnStartupSec=` - this directive is similar to the above timer, but in relation to when the systemd process itself was started.
`OnUnitActiveSec=` - this sets a timer according to when the associated unit was last activated.
`OnUnitInactiveSec=` - this sets the timer in relation to when the associated unit was last marked as inactive.
`OnCalendar=` - this allows you to activate the associated unit by specifying an absolute instead of relative to an event.
`AccuracySec=` - this unit is used to set the level of accuracy with which the timer should be adhered to. By default, the associated unit will be activated within one minute of the timer being reached. The value of this directive will determine the upper bounds on the window in which systemd schedules the activation to occur.
`Unit=` - this directive is used to specify the unit that should be activated when the timer elapses. If unset, systemd will look for a .service unit with a name that matches this unit.
`Persistent=` - if this is set, systemd will trigger the associated unit when the timer becomes active if it would have been triggered during the period in which the timer was inactive.
`WakeSystem=` - setting this directive allows you to wake a system from suspend if the timer is reached when in that state.

## `[Slice]` section

The `[Slice]` section of a unit file actually does not have any .slice unit specific configuration.
Instead, it can contain some resource management directives that are actually available to a number of the units listed above.

# Template unit files

Template unit files are used to create multiple instances of unit based on the template.

Base name of the template unit files:
```
example@.service
```
When instance is created from a template, an instance identifier is placed between the `@` and the period
signifying the start of the unit type. For example: `example@instance1.service`

Specifiers:
`%n` Anywhere where this appears in a template file, the full resulting unit name will be inserted.
`%N` This is the same as the above, but any escaping, such as those present in file path patterns, will be reversed.
`%p` This references the unit name prefix. This is the portion of the unit name that comes before the @ symbol.
`%P` This is the same as above, but with any escaping reversed.
`%i` This references the instance name, which is the identifier following the @ in the instance unit. This is one of the most commonly used specifiers because it will be guaranteed to be dynamic. The use of this identifier encourages the use of configuration significant identifiers. For example, the port that the service will be run at can be used as the instance identifier and the template can use this specifier to set up the port specification.
`%I` This specifier is the same as the above, but with any escaping reversed.
`%f` This will be replaced with the unescaped instance name or the prefix name, prepended with a /.
`%c` This will indicate the control group of the unit, with the standard parent hierarchy of /sys/fs/cgroup/ssytemd/ removed.
`%u` The name of the user configured to run the unit.
`%U` The same as above, but as a numeric UID instead of name.
`%H` The host name of the system that is running the unit.
`%%` This is used to insert a literal percentage sign.

By using the above identifiers in a template file, `systemd` will fill in the correct values when interpreting
the template to create an instance unit.


