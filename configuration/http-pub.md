# HTTP Pub plugin

VerneMQ provides a HTTP REST pub plugin for publishing messages using HTTP/REST. The http_pub plugin accepts HTTP POST requests containing message payloads, and then forwards those messages to the appropriate MQTT subscribers.

The HTTP REST plugin can be used to publish messages from a wider range of devices and platforms, that may not support MQTT natively. Please note, while the plugin can handle a decent amount of request the primary protocol of VerneMQ is MQTT. Whenever possible, it is recommended to use MQTT natively to communicate with VerneMQ. 

## Enabling the plugin

The MQTT bridge plugin (`vmq_http_pub`) is distributed with VerneMQ as an integrated plugin but is not enabled by default. After configuring the bridge as described below, make sure to enable the plugin by setting (`vernemq.conf`):

```text
plugins.vmq_http_pub = on
```

## Configuration
### Bind plugin to HTTP(s) listener
By default the plugin is not bound to any listener. It is recommended to use a dedicated HTTPS listener. For security, reasons the use of HTTPS instead of HTTP is prefered. It is possible to have more than one listener.

```text
listener.https.http_pub = 10.241.15.174:3001
listener.https.http_pub.http_module.vmq_http_pub.auth.mode = apikey
listener.https.http_pub.http_modules = vmq_http_pub
```
This configuration defines an HTTPS listener for an application running on the server at IP address 127.0.0.1 and using port 3001. The listener is used to forward HTTP requests to vmq_http_pub.

Additionally, this configuration sets the authentication method for the vmq_http_pub instance to API key. This means that a valid API key is required to access this instance.

It is important to note that this configuration is only a part of a larger configuration file, and that other settings such as SSL certificates, encryption, protocol versions, etc. may also be defined to improve the security and performance of the HTTPS listener.

### Authenthication and Authorization
The plugin currently supports two authenticatation and authorization modes: "on-behalf-of" and "predefined". "On-behalf-of" means, that the client_id, user and password used for authentication and authorization is part of request (payload). Afterwards, the regular VerneMQ authentication and authorization flows are used. When using "predefined" the client, user, and password is bound to the plugin instance. It is recommended to use "on-behalf-of" and use a seperate client_id, user and password for REST-based clients.

For on-behalf-of authentication use:
```text
listener.https.http_pub.http_modules.vmq_http_pub.mqtt_auth = on-behalf-of
```

For predefined, please use a configuration similar to:
```text
listener.https.http_pub.http_modules.vmq_http_pub.mqtt_auth = predefined
listener.https.http_pub.http_modules.vmq_http_pub.mqtt_auth.user = restUser
listener.https.http_pub.http_modules.vmq_http_pub.mqtt_auth.password = restPasswd
listener.https.http_pub.http_modules.vmq_http_pub.mqtt_auth.client-id = restClient
```


## MQTT Payload
The plugin currently supports three different payload encodings: 
* JSON (regular and base64) in body
* Header parameters, and payload in body
* Query String parameters, and payload in body

Which one to choose is depends on your application.

### JSON
```text
{
	"topic": "testtopic/testtopic1",
	"user": "testuser",
	"password": "test123",
	"qos": 1,
	"retain": false,
	"payload": "this is a payload string",
	"user_properties": [{"a":"b"}]
}
```
In order to allow more complex payload to be encoded as part of the json, the payload itself can be also be base64 encoded. The query string "" has to be used to indicate that the payload is base64 encoded.

### Header parameters
Topic, user, password, qos, retain and user_properties can also be part of the HTTP header. The HTTP body is used for the actual message payload.

### Query String
Topic, user, password, qos and retain flag can also be uurlencoded as part of the query string. The HTTP body is used for the actual message payload.

## Examples

## Metrics
The plugin exposes three metrics:
* The number of messages sent through the REST Publish API
* Number of errors reported by the REST Publish API
* Number of Auth errors reported by the REST Publish API

## Misc Notes
* The plugin allows the authentication and authorization flows to override mountpoint, qos and topic.  
* Currently, the regular (non m5) authentication and authorization flow is used.
* The query string payload does not allow to set user parameters.
