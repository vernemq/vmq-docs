---
description: Configure WebSocket Listeners for VerneMQ.
---

# Websockets

VerneMQ supports the WebSocket protocol out of the box. To be able to open a WebSocket connection to VerneMQ, you have to configure a WebSocket listener or Secure WebSocket listener in the `vernemq.conf` file first:

```text
listener.ws.default = 127.0.0.1:9001

listener.wss.wss_default = 127.0.0.1:9002
# To use WSS, you'll have to configure additional options for your WSS listener (called `wss_default` here):
listener.wss.wss_default.cafile = ./etc/cacerts.pem
listener.wss.wss_default.certfile = ./etc/cert.pem
listener.wss.wss_default.keyfile = ./etc/key.pem
```

Keep in mind that you'll use MQTT-over-WebSocket, so you will need a Javascript library that implements the MQTT client behaviour. We have used the [Eclipse Paho client](https://eclipse.org/paho/clients/js/) as well as [MQTT.js](https://github.com/mqttjs/MQTT.js)

You won't be able to open WebSocket connections on a base URL, always add the `/mqtt` path.

When establishing a WebSocket connection to the VerneMQ MQTT broker, the process begins with an HTTP connection that is then upgraded to WebSocket. This upgrade mechanism means the broker's ability to accept connections can be influenced by HTTP listener settings.

In certain scenarios, such as when connecting from a frontend application, the size of HTTP request headers (including cookies) can exceed the default maximum allowed by VerneMQ. This can lead to a 'HTTP 431 Request Header Fields Too Large' error, preventing the connection from being established.

This behavior is configurable in the `vernemq.conf` file to accommodate larger headers:

```text
listener.http.default.max_request_line_length=32000
listener.http.default.max_header_value_length=32000
```