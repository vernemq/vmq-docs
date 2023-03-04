# HTTP Pub plugin

VerneMQ provides a HTTP REST pub plugin for publishing messages using HTTP/REST. The http_pub plugin accepts HTTP POST requests containing message payloads, and then forwards those messages to the appropriate MQTT subscribers.

The HTTP REST plugin can be used to publish messages from a wider range of devices and platforms, that may not support MQTT natively. 

## Enabling the bridge functionality

The MQTT bridge plugin (`vmq_http_pub`) is distributed with VerneMQ as an integrated plugin but is not enabled by default. After configuring the bridge as described below, make sure to enable the plugin by setting (`vernemq.conf`):

```text
plugins.vmq_http_pup = on
```

## Configuration
