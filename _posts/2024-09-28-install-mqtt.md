---
title: "Install MQTT"
date: 2024-09-28
categories: [Engineering,Linux]
tags: [mqtt]
---

System: Debian GNU/Linux 12 (bookworm)

# Install

Install broker:
```
$ sudo apt update
$ sudo apt install mosquitto mosquitto-clients
$ sudo systemctl enable mosquitto
$ mosquitto_ctrl -v
mosquitto_ctrl is a tool for administering certain Mosquitto features.  
mosquitto_ctrl version 2.0.11 running on libmosquitto 2.0.11.
...
```

# Configure

Add users:
```shell
$ sudo mosquitto_passwd -c /etc/mosquitto/.passwd user1
...
$ sudo mosquitto_passwd /etc/mosquitto/.passwd user2
...
```

Create configuration file:
```shell
$ sudo vim /etc/mosquitto/conf.d/auth.conf
listener 1883
allow_anonymous false
password_file /etc/mosquitto/.passwd
```

Check sending a message to remote server:
```
$ sudo apt install mosquitto-clients
# Subscribe
$ mosquitto_sub -h <server> -t test -u <user> -P <pass>
# Publish
$ mosquitto_pub -h <server> -u <user> -P <pass> -t test -m "Hello"
```

Apply configuration file:
```shell
$ sudo systemctl restart mosquitto
$ netstat -ln | grep 1883
tcp        0      0 0.0.0.0:1883            0.0.0.0:*               LISTEN       tcp6       0      0 :::1883                 :::*                    LISTEN
```

## Enable websockets

Create configuration file:
```shell
$ sudo vim /etc/mosquitto/conf.d/websockets.conf
listener 8083
protocol websockets
certfile /etc/letsencrypt/live/msqt.shapehost.io/fullchain.pem
cafile /etc/letsencrypt/live/msqt.shapehost.io/chain.pem
keyfile /etc/letsencrypt/live/msqt.shapehost.io/privkey.pem
```

Apply configuration file:
```shell
sudo systemctl restart mosquitto
```

## Enable SSL/TLS certs

Generate the dhparam certificate:
```shell
$ sudo openssl dhparam -out /etc/mosquitto/certs/dhparam.pem 2048
```


```shell
$ sudo chown -R mosquitto: /etc/mosquitto/certs
$ sudo vim /etc/mosquitto/conf.d/ssl.conf
listener 8883
certfile /etc/letsencrypt/live/msqt.shapehost.io/fullchain.pem
cafile /etc/letsencrypt/live/msqt.shapehost.io/chain.pem
keyfile /etc/letsencrypt/live/msqt.shapehost.io/privkey.pem
dhparamfile /etc/mosquitto/certs/dhparam.pem
```

Apply configuration file:
```shell
$ sudo systemctl restart mosquitto
```
