---
description: >-
  As well as being available as packages that can be installed directly into the
  operating systems, VerneMQ is also available as a Docker image. Below is an
  example of how to set up a couple of VerneMQ
---

# Running VerneMQ using Docker

## Start a VerneMQ cluster node

```text
docker run --name vernemq1 -d erlio/docker-vernemq
```

Somtimes you need to configure a forwarding for ports \(on a Mac for example\):

```text
docker run -p 1883:1883 --name vernemq1 -d erlio/docker-vernemq
```

This starts a new node that listens on 1883 for MQTT connections and on 8080 for MQTT over websocket connections. However, at this moment the broker won't be able to authenticate the connecting clients. To allow anonymous clients use the `DOCKER_VERNEMQ_ALLOW_ANONYMOUS=on` environment variable.

```text
docker run -e "DOCKER_VERNEMQ_ALLOW_ANONYMOUS=on" --name vernemq1 -d erlio/docker-vernemq
```

## Autojoining a VerneMQ cluster

This allows a newly started container to automatically join a VerneMQ cluster. Assuming you started your first node like the example above you could autojoin the cluster \(which currently consists of a single container 'vernemq1'\) like the following:

```text
docker run -e "DOCKER_VERNEMQ_DISCOVERY_NODE=<IP-OF-VERNEMQ1>" --name vernemq2 -d erlio/docker-vernemq
```

\(Note, you can find the IP of a docker container using `docker inspect <CONTAINER_NAME> | grep \"IPAddress\"`\).

## Checking cluster status

To check if the above containers have successfully clustered you can issue the `vmq-admin` command:

```text
docker exec vernemq1 vmq-admin cluster show
+--------------------+-------+
|        Node        |Running|
+--------------------+-------+
|VerneMQ@172.17.0.151| true  |
|VerneMQ@172.17.0.152| true  |
+--------------------+-------+
```

