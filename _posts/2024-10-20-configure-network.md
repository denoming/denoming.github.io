---
title: "Configure network"
date: 2024-10-20
categories: [Engineering,Linux]
tags: [network,avahi,mdns]
---

# Configure

By default `/etc/network/interfaces` file and `/etc/network/interfaces.d` dir are used to configure network.

Usually, file `/etc/network/interfaces` contains:
```text
auto <some-interface>
iface <some-interface> inet dhcp
```

To enable networkd (systemd-networkd):
```shell
# Backup
$ sudo mv /etc/network/interfaces /etc/network/interfaces.save
$ sudo mv /etc/network/interfaces.d /etc/network/interfaces.d.save
# Start networkd service
$ sudo systemctl enable systemd-networkd
$ ip link
...
2: ens18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
```
Out network interface is `ens18`.

Configure `lan0` network:
```shell
$ vim /etc/systemd/network/lan0.network
[Match]
Name=ens18

[Network]
DHCP=ipv4
$ sudo reboot
$ sudo networkctl list
IDX LINK  TYPE     OPERATIONAL SETUP     
  1 lo    loopback carrier     unmanaged
  2 ens18 ether    routable    configured
```

See https://wiki.debian.org/SystemdNetworkd for additional information.

#  Misc

## Configure resolved service

Install:
```shell
$ sudo apt install systemd-resolved
```

Configure:
```shell
$ sudo vim /etc/systemd/resolved.conf
DNS=<local network DNS server address, e.g. 192.168.1.1>
FallbackDNS=8.8.8.8 1.1.1.1
$ sudo systemctl enable systemd-resolved
$ sudo systemctl start systemd-resolved
```

Check:
```shell
$ resolvectl status
Global
          Protocols: +LLMNR +mDNS -DNSOverTLS DNSSEC=no/unsupported
   resolv.conf mode: stub
         DNS Servers 192.168.1.1
Fallback DNS Servers 8.8.8.8 1.1.1.1

Link 2 (ens18)
Current Scopes: DNS LLMNR/IPv4 LLMNR/IPv6
     Protocols: +DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
   DNS Servers: 192.168.1.1
```

Using:
```shell
$ sudo resolvectl query <some-name>.local
```

## Configure avahi service

Install avahi service:
```shell
$ sudo apt install avahi-daemon avahi-utils
```

Enable avahi service:
```shell
$ systemctl enable avahi-daemon.service avahi-daemon.socket
$ systemctl start avahi-daemon.service avahi-daemon.socket
```

Disable IPv6:
```shell
$ sudo vim /etc/avahi/avahi-daemon.conf
...
use-ipv6=no
...
```

Add config for network service: 
```shell
$ vim /etc/avahi/services/z2m.service
<service-group>
  <name replace-wildcards="yes">Zeegbee2MQTT on %h</name>
  <service protocol="ipv4">
    <host-name>misc.local</host-name>
    <type>_http._tcp</type>
    <port>8083</port>
  </service>
</service-group>
```

Add config for host:
```
$ vim /etc/avahi/hosts
192.168.1.40 hv.local
$ avahi-resolve -n hv.local
```

Useful commands:
```
$ avahi-resolve -n misc.local
$ avahi-browse -art | less
# List supported DNS-SD service types
$ avahi-browse --dump-db
# List supported DNS-SD service types (abbreviations)
$ avahi-browse --dump-db -k
```
