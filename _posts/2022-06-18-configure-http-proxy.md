---
title: "Configure HTTP/HTTPS proxy"
date: 2022-06-18
categories: [Engineering,Linux]
tags: [http,https,proxy]
---

# Install

```shell
$ wget https://snapshots.mitmproxy.org/8.1.0/mitmproxy-8.1.0-linux.tar.gz
$ tar xf mitmproxy-8.1.0-linux.tar.gz -C $HOME/.local/bin
$ echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshenv
$ source ~/.zshenv
$ rehash
```

# Run

```shell
$ mitmproxy
```

# Configure certificate

## Install certificate

* Open http://mitm.it/cert/pem;
* Download certificate for Linux and install:
```shell
$ sudo mv mitmproxy-ca-cert.pem /usr/local/share/ca-certificates/mitmproxy.crt
$ sudo update-ca-certificates
```

## Install certificate for browser

* Open http://mitm.it/cert/pem
* Download certificate for Firefox
* Open Chrome -> Certificates -> Authorities -> Import
* Check "Trust this certificate for identifying websites"
* Click Ok

__Note:__ If certificate already exists. Find it and check option mentioned in fourth step.

# Configure proxy

* Click on the network icon (bottom right corner);
* Open "Network Settings";
* Select "Network proxy";
* Write down the host name "localhost" to the "HTTP Proxy" and "HTTPS Proxy" fields;
* Left the value of port field as "8080";
* Off/On network connection;
* Restart browser.

# Check proxy

```shell
$ curl --proxy http://localhost:8080 --trace $HOME/Downloads/wttr.trace "http://wttr.in/Kyiv?0"
```

