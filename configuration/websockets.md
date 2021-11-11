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

