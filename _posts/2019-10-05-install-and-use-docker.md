---
title:  "Install and use Docker on Ubuntu"
date:   2019-10-05
categories: [Engineering, Docker]
tags: [ubuntu,docker]
---

# Install

## Ubuntu

Install:
```shell
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
```

In case of any problems please follow official [instruction](https://docs.docker.com/install/linux/docker-ce/ubuntu).

# Post-Install

## Manage Docker as a non-root user

Post-Install:
```shell
$ sudo groupadd docker
$ sudo usermod -aG docker ${USER}
$ su - ${USER}
$ id -nG
```
Log out and log back in so that your group membership is re-evaluated.

Verify that you can run `docker` commands without `sudo`:
```shell
$ docker run hello-world
```
In case of any problems please follow official [instruction](https://docs.docker.com/install/linux/linux-postinstall).

## Credentials

### Pass

Create GPG key to use for encrypting a password storage:
```shell
$ gpg --generate-key
<username>
```

Setup password storage:
```shell
$ sudo pamac install pass
$ pass init "<username>"
```

Configure docker credentials (choose the last available release [here](https://github.com/docker/docker-credential-helpers/releases)):
```shell
$ wget -P $HOME/.local/bin https://github.com/docker/docker-credential-helpers/releases/download/v0.8.2/docker-credential-pass-v0.8.2.linux-amd64
$ cp docker-credential-pass-v0.8.2.linux-amd64 $HOME/.local/bin/docker-credential-pass
$ chmod +x $HOME/.local/bin/docker-credential-pass
$ tee ~/.docker/config.json > /dev/null <<EOF
{
  "credsStore": "pass",
  "auths": {}
}
EOF
```

Login to docker:
```shell
$ docker login -u <username>
```

### Secret service

```shell
$ mkdir -p $HOME/.local/bin 
$ wget -O $HOME/.local/bin/docker-credential-secretservice https://github.com/docker/docker-credential-helpers/releases/download/v0.9.3/docker-credential-secretservice-v0.9.3.linux-amd64
$ chmod +x $HOME/.local/bin/docker-credential-secretservice
$ vim ~/.docker/config.json
{
  "credsStore": "secretservice",
  "auths": {}
}
$ docker login -u <username>
```


## Multi-architecture build

Configure containerd image store:
```shell
$ sudo vim /etc/docker/daemon.json
{
  "features": {
    "containerd-snapshotter": true
  }
}
$ sudo systemctl restart docker
$ docker info -f '{{ .DriverStatus }}'
[[driver-type io.containerd.snapshotter.v1]]
```
Configure custom builder:
```shell 
$ docker buildx create \
  --name bender \
  --driver docker-container \
  --bootstrap --use
```
# Commands

Build container:
``` shell
$ docker build -t ${USER}/<name> -f <dockerfile> .
```

Run container:
```shell
$ docker run -p 49160:8080 -d ${USER}/<name>
```

List of containers:
```shell
$ docker ps
```

Display container logs:
```shell
$ docker logs <id>
```

Enter to container:
```shell
$ docker exec -it <id> /bin/bash
```

Stop container:
```shell
$ docker stop <id>
```

Remove container:
```shell
$ docker rm <id> # Remove container by id
$ docker rm $(docker ps -a -q -f status=exited) # Remove containers with certain status
$ docker container prune # Remove all stopped containers
```

# Useful links:
* [Get started](https://docs.docker.com/get-started/)
* [Command line](https://docs.docker.com//engine/reference/commandline/docker)

