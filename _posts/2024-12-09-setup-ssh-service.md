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

### Scenario 2: Configure reverse SSH

Given:
* **IP1** - "white" IP address 
* **Port** - SSH server port

Enable gateway ports on server (jump host):
```
$ sudo tee > /dev/null /etc/ssh/sshd_config.d/enable-gateway-ports.conf <<EOF
GatewayPorts yes
EOF
$ sudo systemctl restart ssh
```

Set up key-based authentication on the client (on client):
```
$ ssh-keygen -t ed25519 -f ~/.ssh/tunnel
$ ssh-copy-id -i ~/.ssh/tunnel.pub -p <Port> <user>@<IP1>
```

Check connection:
```
$ sudo apt install autossh -y
$ autossh -M 0 -o "ServerAliveInterval 30" -o "ServerAliveCountMax 3" -N -R 2222:localhost:22 -p <Port> <user>@<IP1> -i ~/.ssh/tunnel
# On jump host we shall observer opened port 2222
```

Set systemd configuration:
```
$ sudo tee > /dev/null /etc/systemd/system/reverse-tunnel.service <<EOF
[Unit]
Description=AutoSSH Reverse Tunnel to Public Server
After=network-online.target
Wants=network-online.target

[Service]
User=root
Environment="AUTOSSH_GATETIME=0"
ExecStart=/usr/bin/autossh -M 0 -o "ServerAliveInterval 30" -o "ServerAliveCountMax 3" -N -R 2222:localhost:22 -i ~/.ssh/tunnel -p <Port> <user>@<IP1>
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now reverse-tunnel.service
```

### Scenario 3: Configure SSH jumping

Connect to host destination through the chain of one or more jump hosts.

```shell
$ ssh -p <custom-port> -i <key> -J <jump-host-1>,<jump-host-2>,... <user>@<ip-address>
```

Note: Use ~/.ssh/config to specify configuration for jump hosts

### Scenario 4: Local TCP forwarding

```shell
$ ssh -N -L <local-port>:<remote-server>:<remote-port> <ssh-server>
```

Provides forwarding from localhost: from `<local-port>` to `remote-server>:<remote-port>` through SSH server. The `<remote-server>` shell be accessible by `<ssh-server>`.

On remote server TCP forwarding have to be actived:
```
$ vim /etc/ssh/sshd_config
AllowTcpForwarding yes
```

### Scenario 5: Local TCP forwarding

```shell
$ sudo apt install -y pv
$ yes | pv | ssh <remote-host> "cat > /dev/null" 
```

### Scenario 6: Local TCP forwarding

```shell
$ ssh -f -C2qTnN -D <listenting-port> <user>@<ssh-server>
```

### Scenario 7: Disable host key checking

During connecting:
```
$ ssh -o StrictHostKeyChecking=no <address>
```

In config file `~/.ssh/config`:
```
Host 192.168.1.*
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
```
