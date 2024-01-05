---
title: "Install Syncthing on CentOS"
date: 2021-08-31
categories: [Development,Tools]
tags: [syncthing,tools]
---

# Information

* OS version - CentOS Linux release 7.3.1611 (Core)
* Syncthing version - 0.12.22

Create service user and configure environment:
```bash
$ sudo useradd -c "Synchronizer User Account" -M -d /nonexistent syncthing
$ sudo mkdir /var/syncthing
$ sudo chown syncthing:syncthing syncthing
$ sudo usermod -d /var/syncthing syncthing
```

Download and install syncthing:
```bash
$ sudo su - syncthing
$ mkdir bin storage config
$ wget https://github.com/syncthing/syncthing/releases/download/v0.12.22/syncthing-linux-amd64-v0.12.22.tar.gz
$ tar xf syncthing-linux-amd64-v0.12.22.tar.gz
$ rm -f syncthing-linux-amd64-v0.12.22.tar.gz
$ cp syncthing-linux-amd64-v0.12.22/syncthing ~/bin/syncthing
```

Configure Syncthing:
```bash
$ ~/bin/syncthing -generate ~/config
$ nano ~/config/config.xml
...
# Change address
<gui enabled="true" tls="false">
    <address>0.0.0.0:8080</address>
</gui>
$ sudo usermod -s /usr/sbin/nologin syncthing
```

Configure internet access through firewall:
```bash
$ sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent
$ sudo firewall-cmd --zone=public --add-port=22000/tcp --permanent
$ sudo firewall-cmd --reload
```

Configure systemd unit-file:
```bash
$ sudo cp syncthing-linux-amd64-v0.12.22/etc/linux-systemd/system/syncthing-resume.service /etc/systemd/system
$ sudo cp syncthing-linux-amd64-v0.12.22/etc/linux-systemd/system/syncthing@.service /etc/systemd/system
$ sudo chown root:root /etc/systemd/system/syncthing-resume.service
$ sudo chown root:root /etc/systemd/system/syncthing@.service
$ sudo nano /etc/systemd/system/syncthing@.service
[Unit]
Description=Syncthing - Open Source Continuous File Synchronization for %I
Documentation=man:syncthing(1)
After=network.target
Wants=syncthing-inotify@.service

[Service]
User=%i
ExecStart=/var/syncthing/bin/syncthing -no-browser -no-restart -home=/var/syncthing/config -logflags=0
Restart=on-failure
SuccessExitStatus=3 4
RestartForceExitStatus=3 4

[Install]
WantedBy=multi-user.target
$ sudo systemctl start syncthing@syncthing
$ sudo systemctl status syncthing@syncthing
$ sudo systemctl enable syncthing@syncthing
```

Access to `https://<host>:8080` and finalize configuration.
