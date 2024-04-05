
<!-- TOC -->

Command Line Options:

| Option       | Description                            | Option      | Description                                              |
|--------------|----------------------------------------|-------------|----------------------------------------------------------|
| `-A`         | Print frame payload in ASCII           | `-q`        | Quick output (less protocol information)                 |
| `-c <count>` | Exit after capturing **count** packets | `-r <file>` | Read packets from file                                   |
| `-D`         | List available interfaces              | `-s <len>`  | Capture up to len bytes per packet                       |
| `-e`         | Print link-level headers               | `-S`        | Print absolute TCP sequence numbers                      |
| `-F <file>`  | Use file as the filter expression      | `-t`        | Don't print timestamps                                   |
| `-G <n>`     | Rotate the dump file every n seconds   | `-v[v[v]]`  | Print more verbose output                                |
| `-i <iface>` | Specifies the capture interface        | `-w <file>` | Write captured packets to file                           |
| `-K`         | Don't verify TCP checksums             | `-x`        | Print frame payload in hex                               |
| `-L`         | List data link types for the interface | `-X[X]`     | Print frame payload in hex and ASCII (+ ethernet header) |
| `-n`         | Don't convert addresses to names       | `-y <type>` | Specify the data link type                               |
| `-p`         | Don't capture in promiscuous mode      | `-Z <user>` | Drop privileges from root to use                         |



Capture Filter Primitives:

| Expression                                      | Description                                                   |
|-------------------------------------------------|---------------------------------------------------------------|
| `src or dst host <host>`                        | Matches a host as the IP source, destination, or either       |
| `ether [src or dst] host <ehost>`               | Matches a host as the Ethernet source, destination, or either |
| `gateway host <host>`                           | Matches packets which used host as a gateway                  |
| `[src or dst] net <network>/<len>`              | Matches packets to or from an endpoint residing in network    |
| `[tcp or udp] [src or dst] port <port>`         | Matches TCP or UDP packets sent to/from port                  |
| `[tcp or udp] [src or dst] portrange <p1>-<p2>` | Matches TCP or UDP packets to/from a port in the given range  |
| `less <length>`                                 | Matches packets less than or equal to length                  |
| `greater <length>`                              | Matches packets greater than or equal to length               |
| `(ether or ip or ip6) proto <protocol>`         | Matches an Ethernet, IPv4, or IPv6 protocol                   |
| `(ether or ip) broadcast`                       | Matches Ethernet or IPv4 broadcas                             |
| `(ether or ip or ip6) multicast`                | Matches Ethernet, IPv4, or IPv6 multicasts                    |
| `type (mgt or ctl or data) [subtype <subtype>]` | Matches 802.11 frames based on type and optional subtype      |
| `vlan [<vlan>]`                                 | Matches 802.1Q frames, optionally with a VLAN ID of vlan      |
| `mpls [<label>]`                                | Matches MPLS packets, optionally with a label of label        |
| `<expr> <relop> <expr>`                         | Matches packets by an arbitrary expression                    |


Protocols:

| Protocol | Protocol | Protocol |
|----------|----------|----------|
| arp      | ip6      | slip     |
| ether    | link     | tcp      |
| fddi     | ppp      | tr       |
| icmp     | radio    | udp      |
| ip       | rarp     | wlan     |

Modifiers:

| Modifier      | Example                          | Description                 |
|---------------|----------------------------------|-----------------------------|
| `! or not`    | `udp dst port not 53`            | UDP not bound for port 53   |
| `&& or and`   | `host 10.0.0.1 && host 10.0.0.2` | Traffic between these hosts |
| `\|\| or or`  | `tcp dst port 80 or 8080`        | Packets to either TCP port  |

TCP Flags:

| Flag    | Flag    |
|---------|---------|
| tcp-urg | tcp-rst |
| tcp-ack | tcp-syn |
| tcp-psh | tcp-fin |

ICMP Types:

| Type              | Type               | Type             |
|-------------------|--------------------|------------------|
| icmp-echoreply    | icmp-routeradvert  | icmp-tstampreply |
| icmp-unreach      | icmp-routersolicit | icmp-ireq        |
| icmp-sourcequench | icmp-timxceed      | icmp-ireqreply   |
| icmp-redirect     | icmp-paramprob     | icmp-maskreq     |
| icmp-echo         | icmp-tstamp        | icmp-maskreply   |


* Capture traffic by given interface or all interfaces
```shell
$ tcpdump -i <interface>|any
```

* Capture HTTPS traffic
```shell
$ tcpdump [-c 1|n] -nnSX port 443
-c 1|n - specify particular number of HTTPS packets to capture
```

* Capture traffic by host and network:
```shell
$ tcpdump [src|dst] (host|net) <IP address / network>
```

* Capture traffic by protocol:
```shell
$ tcpdump -X (icmp|igmp|igrp|ip|ip6|ipv6|rarp|rip|sccp|tcp|udp)
```

* Capture credentials in plain text
```shell
$ tcpdump -A -i <interface> 'port http or port ftp or port telnet' | grep -i 'user\|pass\|login'
or
$ tcpdump port http or port ftp or port smtp or port imap or port pop3 or port telnet -lA | \
  egrep -i -B5 'pass=|pwd=|log=|login=|user=|username=|pw=|passw=|passwd=|password=|pass:|user:|username:|password:|login:|pass |user ' 
```

* Capture traffic to a known malicious domain or an untrusted site:
```shell
$ tcpdump -i <interface> dst host <some-site.com>
```

* Capture traffic from a specific IP range:
```shell
$ tcpdump src net <IP range of country> (e.g. 217.117.79.0/24)
```

* Capture SMB traffic:
```shell
$ tcpdump -nn -i <interface> port 445
```

* Capture TCP RESET/ACK packets:
```shell
$ tcpdump 'tcp[13] = 41'
```

* Capture traffic based on packet size:
```shell
$ tcpdump 'len > 32 and len < 64'
```

* Capture traffic with raw output view:
```shell
$ tcpdump -ttnnvvS
```

* Capture all traffic from specific IP and destined to a specific port:
```shell
$ tcpdump -nnvvS src <IP address> and dst port <port>
```

* Capture all traffic from one network to another:
```shell
$ tcpdump -nvX src net <address1/len1> and dst net <address2/len2> or <address3/len3>
```

* Capture traffic going to specific IP excluding ICMP
```shell
$ tcpdump dst <IP address> and src net and not icmp
```

* Capture traffic from given host that isn't on a specific port:
```shell
$ tcpdump -vv src mars and not dst port 22
```

* Capture TCP flags:
```shell
# TCP RST flags
$ tcpdump 'tcp[13] & 4!=0'tcpdump 'tcp[tcpflags]  == tcp-rst'
# TCP SYN flags
$ tcpdump 'tcp[13] & 2!=0'tcpdump 'tcp[tcpflags]  == tcp-syn'
# SYN and ACK flags
$ tcpdump 'tcp[13]=18'
# TCP URG flags
$ tcpdump 'tcp[13] & 32!=0'tcpdump 'tcp[tcpflags]  == tcp-urg'
# TCP ACK flags
$ tcpdump 'tcp[13] & 16!=0' tcpdump 'tcp[tcpflags]  == tcp-ack'
# TCP PSH flags
$ tcpdump 'tcp[13] & 8!=0' tcpdump 'tcp[tcpflags]  == tcp-push'
# TCP FIN flags
$ tcpdump 'tcp[13] & 1!=0'tcpdump 'tcp[tcpflags]  == tcp-fin'
```

* Capture HTTP traffic with grepping:
```shell
# Find HTTP user agents
$ tcpdump -vvAls0 | grep 'User-Agent:'
# GET Requests
$ tcpdump -vvAls0 | grep 'GET'
# Find HTTP host headers
$ tcpdump -vvAls0 | grep 'Host:'
# Find HTTP cookies
$ tcpdump -vvAls0 | grep 'Set-Cookie|Host:|Cookie:'
```

* Capture protocol specific traffic:
```shell
# Find SSH connections
$ tcpdump 'tcp[(tcp[12]>>2):4] = 0x5353482D'
# Find DNS traffic
$ tcpdump -vvAs0 port 53
# Find FTP Traffic
$ tcpdump -vvAs0 port ftp or ftp-data
# Find NTP Traffic
$ tcpdump -vvAs0 port 123
```
