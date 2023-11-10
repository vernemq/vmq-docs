---
description: Certificate management
---

# Certificates
VerneMQ supports different Transport Layer Security (TLS) options, which allow for secure communication between MQTT clients and VerneMQ. Certificates typically have only a limited validity (for example one year) after which they have to be replaced. VerneMQ allows to replace a certificate without interrupting active connections.

## Replace a certificate
Replacing a certificate straightforward. One just need to replace (overwrite) the corresponding PEM files. VerneMQ will pickup the new certificates.

For example, if you have the following configuration 

```text
listener.ssl.cafile = /etc/ssl/cacerts.pem
listener.ssl.certfile = /etc/ssl/cert.pem
listener.ssl.keyfile = /etc/ssl/key.pem

listener.ssl.default = 127.0.0.1:8883
```

the files cacerts.pem, cert.pem and key.pem can be overwritten (on the filesystem!) with new certificates. VerneMQ will pick-up the certificate after some time (by default around 2min). It is possible to invalidate the certificate immedialtly by issuing the following command

```text
vmq-admin tls clear-pem-cache
```

One can use the openssl s_client tool to verify that that the new certificate has been deployed:
```text
openssl s_client -host 127.0.0.1 -port 8883
```


## Running sessions and certificate validity
Unless the client is implemented otherwise, all active connection will remain active. Please note, that TCP is designed in a way that the validity is checked during the TLS/SSL handshake, which happens once at the beginning of the session. Running sessions are not affected by an expired certificate. 

In case you want to invalidate all existing connections it is recommended to stop/start the listener. 

```text
vmq-admin listener stop <your listener>
vmq-admin listener start <your listener>
```

If you generally want to force your clients to reconnect after a specified period of time you can configure a maximum connection lifetime, after which a client is disconnected by the broker.

```text
listener.ssl.default.max_connection_lifetime = 25000
```

