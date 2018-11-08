---
description: VerneMQ supports multiple ways to configure one or many MQTT listeners.
---

# MQTT Listeners

Listeners specify on which IP address and port VerneMQ should accept new incoming connections. Depending on the chosen transport \(TCP, SSL, WebSocket\) different configuration parameters have to be provided. VerneMQ allows to write the listener configurations in a hierarchical manner, enabling very flexible setups. VerneMQ applies reasonable defaults on the top level, which can be of course overridden if needed.

```text
# defines the default nr of allowed concurrent 
# connections per listener
listener.max_connections = 10000

# defines the nr. of acceptor processes waiting
# to concurrently accept new connections
listener.nr_of_acceptors = 10

# used when clients of a particular listener should
# be isolated from clients connected to another 
# listener.
listener.mountpoint = off
```

These are the only default parameters that are applied for all transports, and the only one that are of interest for plain TCP and WebSocket listeners.

These global defaults can be overridden for a specific transport protocol `listener.tcp.CONFIG = VAL`, or even for a specific listener `listener.tcp.LISTENER.CONFIG = VAL`. The placeholder `LISTENER` is freely chosen and is only used as a reference for further configuring this particular listener.

### Allowed protocol versions

Since VerneMQ 1.5.0 it is possible to configure which MQTT protocol versions as listener will accept.

VerneMQ supports MQTT 3.1, 3.1.1, and 5.0 \(since VerneMQ 1.6.0\). To allow these protocol versions, set:

```text
listener.tcp.allowed_protocol_versions = 3,4,5
```

Here `3,4,5` are the protocol level versions corresponding to MQTT 3.1, 3.1.1 and 5.0 respectively. The default value is `3,4` thus allowing MQTT 3.1 and 3.1.1, while MQTT 5.0 is disabled.

### Sample Config

Listen on TCP port 1883 and for WebSocket Connections on port 8888:

```text
listener.tcp.default = 127.0.0.1:1883
listener.ws.default = 127.0.0.1:8888
```

An additional listener can be added by using a different **name**. In the example above the **name** equals to `default` and can be used for further configuring this particular listener. The following example demonstrates how an additional listener is defined as well as how the maximum number of connections can be limited for this listener:

```text
listener.tcp.my_other = 127.0.0.1:18884
listener.tcp.my_other.max_connections = 100
```

### PROXY protocol

VerneMQ listeners can be configured to accept connections from a proxy server that supports the PROXY protocol. This enables VerneMQ to retrieve peer information such as source IP/Port but also PROXY Version 2 protocol TLS client certificate details if the proxy was used to terminate TLS.

To enable the PROXY protocol for tcp listeners use `listener.tcp.proxy_protocol = on` or for a specific listener use `listener.tcp.LISTENER.proxy_protocol = on`.

### Sample SSL Config

Accepting SSL connections on port 8883:

```text
listener.ssl.cafile = /etc/ssl/cacerts.pem
listener.ssl.certfile = /etc/ssl/cert.pem
listener.ssl.keyfile = /etc/ssl/key.pem

listener.ssl.default = 127.0.0.1:8883
```

If you want to use client certificates to authenticate your clients you have to set the following option:

```text
listener.ssl.require_certificate = on
```

If you use client certificates and want to use the certificates CN value as a username you can set:

```text
listener.ssl.use_identity_as_username = on
```

Both options `require_certificate` and `use_identity_as_username` default to `off`.

The same configuration options can be used for securing WebSocket connections, just use `wss` as the protocol identifier e.g. `listener.wss.require_certificate`.

{% hint style="info" %}
With SSL, you still need to configure authentication and authorization! That is, set `allow_anonymous` to `off`, and configure vmq\_acl and vmq\_passwd or your authentication plugin.

The default listener `listener.vmq.clustering` is used for distributing MQTT messages among the cluster nodes.
{% endhint %}

