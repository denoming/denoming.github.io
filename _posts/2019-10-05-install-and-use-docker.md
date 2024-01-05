---
title:  "Install and use Docker on Ubuntu"
date:   2019-10-05
categories: [Engineering, Docker]
tags: [ubuntu,docker]
---

# Install

~~~ bash
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
~~~

In case of any problems please follow official [instruction](https://docs.docker.com/install/linux/docker-ce/ubuntu).

# Post-Install

~~~ bash
$ sudo groupadd docker
$ sudo usermod -aG docker ${USER}
$ su - ${USER}
$ id -nG
~~~
Log out and log back in so that your group membership is re-evaluated.

Verify that you can run `docker` commands without `sudo`.
~~~ bash
$ docker run hello-world
~~~

In case of any problems please follow official [instruction](https://docs.docker.com/install/linux/linux-postinstall).

# Useful commands
* Build container
~~~ bash
$ docker build -t ${USER}/<name> -f <dockerfile> .
~~~
* Run container
~~~ bash
$ docker run -p 49160:8080 -d ${USER}/<name>
~~~
* List of containers:
~~~ bash
$ docker ps
~~~
* Display container logs:
~~~ bash
$ docker logs <id>
~~~
* Enter to container:
~~~ bash
$ docker exec -it <id> /bin/bash
~~~
* Stop container:
~~~ bash
$ docker stop <id>
~~~
* Remove container:
~~~ bash
$ docker rm <id> # Remove container by id
$ docker rm $(docker ps -a -q -f status=exited) # Remove containers with certain status
$ docker container prune # Remove all stopped containers
~~~



# Useful links:
* [Get started](https://docs.docker.com/get-started/)
* [Command line](https://docs.docker.com//engine/reference/commandline/docker)

