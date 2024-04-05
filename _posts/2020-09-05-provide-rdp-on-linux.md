---
title: "Provide RDP on Linux machine"
date: 2020-09-05
categories: [Engineering,Linux]
tags: [rdp]
---

## By NoMachine (preferable)

1. From [site](https://www.nomachine.com/download) download server and client;
2. Install server and client;
3. Connect to machine using client;
4. Enjoy.

## By X2Go

Install SSH server:
```bash
$ sudo apt -y install openssh-server
$ sudo ufw allow 22
```

Install MATE Desktop:
```bash
$ sudo apt -y update
$ sudo apt -y install mate-desktop-environment-extras
```

Install X2Go server:
```bash
$ sudo apt-add-repository ppa:x2go/stable
$ sudo apt -y update
$ sudo apt -y install x2goserver x2goserver-xsession
$ sudo apt -y install x2gomatebindings
```