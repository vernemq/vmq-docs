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

## Mountpoints

Normally, an MQTT broker hosts one single topic tree. This means that all topics are accessible to all publishers and subscribers \(limited by the ACLs you configured, of course\). Mountpoints are a way to host multiple topic trees in a single broker. They are completely separated and clients with different topic trees cannot publish messages to each other. This could be useful if you provide MQTT services to multiple separated use cases/verticals or clients, with a single broker. Note that mountpoints are configured via different listeners. As a consequence, the MQTT clients will have to connect to a specific port to connect to a specific topic space \(mountpoint\).

The mountpoints can be configured on the protocol level or configurred or overridden on the specific listener level.

```text
listener.ssl.mountpoint = ssl-mountpoint

listener.tcp.listener1.mountpoint = tcp-listener1
listener.tcp.listener2.mountpoint = tcp-listener2
```

## Allowed protocol versions

Since VerneMQ 1.5.0 it is possible to configure which MQTT protocol versions as listener will accept.

VerneMQ supports MQTT 3.1, 3.1.1, and 5.0 \(since VerneMQ 1.6.0\). To allow these protocol versions, set:

```text
listener.tcp.allowed_protocol_versions = 3,4,5
```

Here `3,4,5` are the protocol level versions corresponding to MQTT 3.1, 3.1.1 and 5.0 respectively. The default value is `3,4` thus allowing MQTT 3.1 and 3.1.1, while MQTT 5.0 is disabled.

## Sample Config

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

## PROXY protocol

VerneMQ listeners can be configured to accept connections from a proxy server that supports the PROXY protocol. This enables VerneMQ to retrieve peer information such as source IP/Port but also PROXY Version 2 protocol TLS client certificate details if the proxy was used to terminate TLS.

To enable the PROXY protocol for tcp listeners use `listener.tcp.proxy_protocol = on` or for a specific listener use `listener.tcp.LISTENER.proxy_protocol = on`.

If client certificates are used you can set `listener.tcp.proxy_protocol_use_cn_as_username = on` which will overwrite the MQTT username set by the client with the common name from the client certificate before authentication and authorization is performed. 

## SSL/TLS Support
VerneMQ supports different Transport Layer Security (TLS) options, which allow for secure communication between MQTT clients and VerneMQ. 

TLS provides secure communication between devices by encrypting the data in transit, preventing unauthorized access and ensuring the integrity of the data. VerneMQ supports various TLS options, including the use of certificates, mutual authentication, Pre-Shared Keys and the ability to specify specific ciphersuites and TLS versions.

VerneMQ supports the following the TLS-flavours:

- Server Side TLS
- TLS-PSK
- Mutal TLS (mTLS)

In server-side TLS, the client initiates a TLS handshake with the broker, and the broker responds by sending its certificate. The client verifies the certificate and generates a symmetric key, which is used to encrypt and decrypt data exchanged between the client and broker. Server-side TLS does no further authentication or authorization of the client. The broker later on authenticates and authorizes clients through MQTT.    

TLS-PSK (Pre-Shared Key) secures communication between MQTT client and broker using pre-shared keys for authentication. Unlike Service-Side or mutal TLS, which use certificates to authenticate the server and client, TLS-PSK uses a pre-shared secret (a key) to authenticate the endpoints. Clients that support TLS-PSK can use the specified pre-shared keys to authenticate themselves to the broker, providing a lightweight alternative to certificate-based authentication. The key has to be securly stored on the MQTT device.

Mutal TLS (mTLS)  provides mutual authentication and encryption of data in transit between MQTT client and Broker. Unlike Server-Side TLS, where only the server is authenticated to the client, mTLS requires both the client and server to authenticate each other before establishing a secure connection.

The decision to use TLS, TLS-PSK, or mTLS depends on your specific use case and security requirements. 

### Sample SSL Config
#### Service-Side TLS
Accepting SSL connections on port 8883:

```text
listener.ssl.cafile = /etc/ssl/cacerts.pem
listener.ssl.certfile = /etc/ssl/cert.pem
listener.ssl.keyfile = /etc/ssl/key.pem

listener.ssl.default = 127.0.0.1:8883
```

#### TLS-PSK (Pre-Shared Keys)
The following configuration snippet enables TLS-PSK authentication on VerneMQ's SSL listener, specifies the location of the pre-shared key file, and sets the list of ciphers to be used for encryption. Clients that support TLS-PSK can use the specified pre-shared keys to authenticate themselves to the broker, providing a lightweight alternative to certificate-based authentication.

```text
listener.ssl.psk_support = on
listener.ssl.pskfile = /srv/vernemq/etc/vernemq.psk
listener.ssl.ciphers  = PSK-AES256-GCM-SHA384:PSK-AES256-CBC-SHA:PSK-AES128-GCM-SHA256
```
This configuration snippet enables TLS-PSK authentication on the VerneMQs SSL listener, specifies the location of the pre-shared key file, and sets the list of ciphers to be used for encryption. Clients that support TLS-PSK can use the specified pre-shared keys to authenticate themselves to the broker, providing a lightweight alternative to certificate-based authentication.

The PSK file contains a list of matching identifiers and psk keys.


#### Mutal TLS (mTLS)
If you want to use client certificates to authenticate your clients you have to set the following option:

```text
listener.ssl.require_certificate = on
```

If you use client certificates and want to use the certificates CN value as a username you can set:

```text
listener.ssl.use_identity_as_username = on
```

Both options `require_certificate` and `use_identity_as_username` default to `off`. mTLS can work with additional MQTT-based authentication (username and password) or without. In case you want to use only mTLS-based authentication you need to enable allow_anonymous (global) or allow_anonymous_override (listener).

```text
listener.ssl.allow_anonymous_override = on
```

### WebSocket
The same configuration options can be used for securing WebSocket connections, just use `wss` as the protocol identifier e.g. `listener.wss.require_certificate`.

{% hint style="info" %}
With SSL, you still need to configure authentication and authorization! That is, set `allow_anonymous` to `off`, and configure vmq\_acl and vmq\_passwd or your authentication plugin.

The default listener `listener.vmq.clustering` is used for distributing MQTT messages among the cluster nodes.
{% endhint %}

