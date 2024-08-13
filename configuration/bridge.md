---
description: VerneMQ can interface with other brokers (and itself) via MQTT bridges.
---

# MQTT Bridge

Bridges are a non-standard way (but de-facto standard) among MQTT broker implementations to connect two different MQTT brokers. Over a bridge, the topic tree of a remote broker becomes part of the topic tree on the local broker. VerneMQ bridges support plain TCP connections as well as SSL connections.

A bridge will be a point-to-point connection between 2 brokers, but can still forward all the messages from all cluster nodes to another cluster.

{% hint style="info" %} The VerneMQ bridge plugin currently forwards messages using MQTT protocol version 3.1.1. MQTT v5 messages will still be forwarded but be aware that metadata like user-defined properties will be dropped.{% endhint %}



## Enabling the bridge functionality

The MQTT bridge plugin (`vmq_bridge`) is distributed with VerneMQ as an integrated plugin but is not enabled by default. After configuring the bridge as described below, make sure to enable the plugin by setting (`vernemq.conf`):

```text
plugins.vmq_bridge = on
```

See [Managing plugins](plugins.md) for more information on working with plugins.

Basic information on the configured bridges can be displayed on the admin CLI:

```text
$ vmq-admin bridge show
+-----------------+-----------+----------+-------------------+
|   endpoint      |buffer size|buffer max|buffer dropped msgs|
+-----------------+-----------+----------+-------------------+
|192.168.1.10:1883|     0     |    0     |         0         |
+-----------------+-----------+----------+-------------------+
```
{% hint style="info" %}
The `vmq-admin bridge` command is only available when the bridge plugin is running.
{% endhint %}

## Sample MQTT Bridge

To configure `vmq_bridge` you need to edit the bridge section of the `vernemq.conf` file to set endpoints and
mapping topics. A brigde can push or pull messages, as defined in the topic pattern list.

Setup a bridge to a remote broker:

```text
vmq_bridge.tcp.br0 = 192.168.1.12:1883
```

Different connection parameters can be set:

```text
# use a clean session (defaults to 'off')
vmq_bridge.tcp.br0.cleansession = off | on

# set the client id (defaults to 'auto', which generates one)
vmq_bridge.tcp.br0.client_id = auto | my_bridge_client_id

# set keepalive interval (defaults to 60 seconds)
vmq_bridge.tcp.br0.keepalive_interval = 60

# set the username and password for the bridge connection
vmq_bridge.tcp.br0.username = my_bridge_user
vmq_bridge.tcp.br0.password = my_bridge_pwd

# set the restart timeout (defaults to 10 seconds)
vmq_bridge.tcp.br0.restart_timeout = 10

# VerneMQ indicates other brokers that the connection
# is established by a bridge instead of a normal client.
# This can be turned off if needed:
vmq_bridge.tcp.br0.try_private = off

# Set the maximum number of outgoing messages the bridge will buffer
# while not connected to the remote broker. Messages published while
# the buffer is full are dropped. A value of 0 means buffering is
# disabled.
vmq_bridge.tcp.br0.max_outgoing_buffered_messages = 100
```

Define the topics the bridge should incorporate in its local topic tree \(by subscribing to the remote\), or the topics it should export to the remote broker \(by publishing to the remote\). We share a similar configuration syntax to that used by the Mosquitto broker:

```text
topic [[[ out | in | both ] qos-level] local-prefix remote-prefix]
```

> `topic` defines a topic pattern that is shared between the two brokers. Any topics matching the pattern \(which may include wildcards\) are shared. The second parameter defines the direction that the messages will be shared in, so it is possible to import messages from a remote broker using `in`, export messages to a remote broker using `out` or share messages in `both` directions. If this parameter is not defined, VerneMQ defaults to `out`. The QoS level defines the publish/subscribe QoS level used for this topic and defaults to `0`. _\(Source: mosquitto.conf\)_

The `local-prefix` and `remote-prefix` can be used to prefix incoming or outgoing publish messages.

{% hint style="warning" %}
Currently the `#` wildcard is treated as a comment from the configuration parser, please use `*` instead.
{% endhint %}

A simple example:

```text
# share messages in both directions and use QoS 1
vmq_bridge.tcp.br0.topic.1 = /demo/+ both 1

# import the $SYS tree of the remote broker and
# prefix it with the string 'remote'
vmq_bridge.tcp.br0.topic.2 = $SYS/* in remote
```

## Sample MQTT Bridge that uses SSL/TLS

SSL bridges support the same configuration parameters as TCP bridges (change `.tcp` to `.ssl`), but need further instructions for handling the SSL specifics:

```text
vmq_bridge.ssl.br0 = 192.168.1.12:1883

# set the username and password for the bridge connection
vmq_bridge.ssl.br0.username = my_bridge_user
vmq_bridge.ssl.br0.password = my_bridge_pwd

# define the CA certificate file or the path to the
# installed CA certificates
vmq_bridge.ssl.br0.cafile = cafile.crt
#or
vmq_bridge.ssl.br0.capath = /path/to/cacerts

# if the remote broker requires client certificate authentication
vmq_bridge.ssl.br0.certfile = /path/to/certfile.pem
# and the keyfile
vmq_bridge.ssl.br0.keyfile = /path/to/keyfile

# disable the verification of the remote certificate (defaults to 'off')
vmq_bridge.ssl.br0.insecure = off

# set the used tls version (defaults to 'tlsv1.2')
vmq_bridge.ssl.br0.tls_version = tlsv1.2
```

## Restarting MQTT Bridges

MQTT Bridges that are initiated from the source broker (push bridges) are started when VerneMQ boots and finds a bridge configuration in the `vernemq.conf` file.
Sometimes it's useful to restart MQTT bridges without restarting a broker. This can be done by disabling, then enabling the `vmq_bridge` plugin and manually calling the `bridge start` command:

```text
$ sudo vmq-admin plugin disable --name vmq_bridge
$ sudo vmq-admin plugin enable --name vmq_bridge
$ sudo vmq-admin bridge start
```
