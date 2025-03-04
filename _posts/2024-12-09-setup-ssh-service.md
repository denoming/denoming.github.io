---
title: "Setup SSH service"
date: 2024-12-09
categories: [Engineering,Linux]
tags: [ssh]
---

# Install

Install SSH service:
```shell
$ sudo apt update
$ sudo apt install openssh-server
```

# Configure

## Scenarios

### Scenario 1: Configure login to remote server by SSH key

Configure SSH key (local host):
```
# Generate SSH Key
$ ssh-keygen -t rsa -a 64 -b 4096 -C "<meaningful-name>" -f $HOME/.ssh/<file-name>
# Copy SSH Key
$ ssh-copy-id -i $HOME/.ssh/<file-name> <user>@<ip-address>
# Try to login
$ ssh -i <file-name> <user>@<ip-address>
```

Config client to use identity key file by default:
```shell
$ vim ~/.ssh/config
Host rpi5  
  HostName=192.168.1.90  
  User=denys  
  IdentityFile=~/.ssh/Denys-PC
$ ssh rpi5
...
```

Configure remote server to allow login by SSH key only:
```
$ vim /etc/ssh/sshd_config
PubkeyAuthentication yes
PasswordAuthentication no
UsePAM no
$ systemctl restart sshd
```

### Scenario 2: Configure SSH Jumping

Connect to host destination throught the chain of one or more jump hosts.

```shell
$ ssh -p <custom-port> -i <key> -J <jump-host-1>,<jump-host-2>,... <user>@<ip-address>
```

Note: Use ~/.ssh/config to specify configuration for jump hosts

### Scenario 3: Local TCP Forwarding

```shell
$ ssh -N -L <local-port>:<remote-server>:<remote-port> <ssh-server>
```

Provides forwarding from localhost: from `<local-port>` to `remote-server>:<remote-port>` through SSH server. The `<remote-server>` shell be accessible by `<ssh-server>`.

On remote server TCP forwarding have to be actived:
```
$ vim /etc/ssh/sshd_config
AllowTcpForwarding yes
```

### Scenario 4: Local TCP Forwarding

```shell
$ sudo apt install -y pv
$ yes | pv | ssh <remote-host> "cat > /dev/null" 
```

### Scenario 5: Local TCP Forwarding

```shell
$ ssh -f -C2qTnN -D <listenting-port> <user>@<ssh-server>
```
