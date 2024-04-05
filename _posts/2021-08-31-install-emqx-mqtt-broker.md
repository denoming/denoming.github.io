---
title: "Install EMQX MQTT Broker"
date: 2021-08-31
categories: [Development,Tools]
tags: [emqx,mqtt,tools]
---

Pull and run by docker:
```bash
$ docker pull emqx/emqx:latest
$ docker run -d --name emqx \
-p 1883:1883   \
-p 8081:8081   \
-p 8083:8083   \
-p 8883:8883   \
-p 8084:8084   \
-p 18083:18083 \
emqx/emqx:latest
```

Enable authentication plugin:
```bash
$ sudo apt install curl jq
# Get node id
$ curl --basic -su admin:public -X GET "http://localhost:8081/api/v4/brokers" | jq .data[0].node
# Enable auth plugin
$ curl --basic -su admin:public \
-X PUT "http://localhost:8081/api/v4/nodes/<node>/plugins/emqx_auth_mnesia/load"
```

Create user:
```bash
# Create test user credentials
$ curl --basic -su admin:public \
-X POST "http://localhost:8081/api/v4/auth_username" -d '{"username": "test", "password": "test"}'
# Check test user credentials
$ curl --basic -su admin:public \
-X GET "http://localhost:8081/api/v4/auth_username/test" | jq
```