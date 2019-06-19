---
description: How to implement VerneMQ plugins using a HTTP interface
---

# Webhooks

The VerneMQ Webhooks plugin provides an easy and flexible way to build powerful plugins for VerneMQ using web hooks. With VerneMQ Webhooks you are free to select the implementation language to match your technical requirements or the language in which you feel comfortable and productive in. You can use any modern language such as Python, Go, C\#/.Net and indeed any language in which you can build something that can handle HTTP requests.

The idea of VerneMQ Webhooks very simple: you can register an HTTP endpoint with a VerneMQ plugin hook and whenever the hook \(such as `auth_on_register`\) is called, the VerneMQ Webhooks plugin dispatches a HTTP post request to the registered endpoint. The HTTP post request contains a HTTP header like `vernemq-hook: auth_on_register` and a JSON encoded payload. The endpoint then responds with code 200 on success and with a JSON encoded payload informing the VerneMQ Webhooks plugin which action to take \(if any\).

## Configuring webhooks

To enable webhooks make sure to set:

```text
plugins.vmq_webhooks = on
```

And then each webhook can be configured like this:

```text
vmq_webhooks.mywebhook1.hook = auth_on_register
vmq_webhooks.mywebhook1.endpoint = http://127.0.0.1/myendpoints
```

It is also possible to dynamically register webhooks at run-time:

```text
$ vmq-admin webhooks register hook=auth_on_register endpoint="http://localhost"
```

See which endpoints are registered:

```text
$ vmq-admin webhooks show
```

And finally deregistering an endpoint:

```text
$ vmq-admin webhooks deregister hook=auth_on_register endpoint="http://localhost"
```

{% hint style="info" %}
We recommend placing the endpoint implementation locally on each VerneMQ node such that each request can go over localhost without being subject to network issues. Also note that currently VerneMQ Webhooks does not encrypt requests in any way or use HTTPS, so care should be taken if the endpoints are made reachable over the network.
{% endhint %}

## Connection pool configuration

Each registered hook uses by default a connection pool containing maximally 100 connections. This can be changed by setting `vmq_webhooks.pool_max_connections` to a different value. Similarly the `vmq_webhooks.pool_timeout` configuration \(value is in milliseconds\) can be set to control how long an unused connection should stay in the connection pool before being closed and removed. The default value is 60000 \(60 seconds\).

These options are available in VerneMQ 1.4.0.

## Caching

VerneMQ webhooks support caching of the `auth_on_register`, `auth_on_publish` and `auth_on_subscribe` hooks.

This can be used to speed up authentication and authorization tremendously. All data passed to these hooks is used to look if the call is in the cache, except in the case of the `auth_on_publish` where the payload is omitted.

To enable caching for an endpoint simply return the `cache-control: max-age=AgeInSeconds` in the response headers to one of the mentioned hooks. If the call was successful \(authentication granted\), the request will be cached together with any modifiers, except for the `payload` modifier in the `auth_on_publish` hook.

Whenever a non-expired entry is looked up in the cache the endpoint will not be called and the modifiers of the cached entry will be returned, if any.

It is possible to inspect the cache using:

```text
$ vmq-admin webhooks cache show
```

{% hint style="info" %}
Cache entries are currently not actively disposed after expiry and will remain in memory.
{% endhint %}

## Webhook specs

All webhooks are called with method `POST`. All hooks need to be answered with the HTTP code `200` to be considered successful. Any hook called that does not return the `200` code will be logged as an error as will any hook with an unparseable payload.

All hooks are called with the header `vernemq-hook` which contains the name of the hook in question.

For detailed information about the hooks and when they are called, see the sections [Session Lifecycle](sessionlifecycle.md), [Subscribe Flow](subscribeflow.md) and [Publish Flow](publishflow.md).

{% hint style="info" %}
Note, when overriding a **mountpoint** or a **client-id** both have to be returned by the webhook implementation for it to have an effect.
{% endhint %}

### auth\_on\_register

Header: `vernemq-hook: auth_on_register`

Webhook example payload:

```javascript
{
    "peer_addr": "127.0.0.1",
    "peer_port": 8888,
    "username": "username",
    "password": "password",
    "mountpoint": "",
    "client_id": "clientid",
    "clean_session": false
}
```

A minimal response indicating the authentication was successful looks like:

```javascript
{
    "result": "ok"
}
```

It is also possible to override various client specific settings by returning an array of modifiers:

```javascript
{
    "result": "ok",
    "modifiers": {
        "max_message_size": 65535,
        "max_inflight_messages": 10000,
        "retry_interval": 20000
    }
}
```

Note, the `retry_interval` is in milli-seconds. It is possible to override many more settings, see the [Session Lifecycle](sessionlifecycle.md) for more information.

Other possible responses:

```javascript
{
  "result": "next"
}
```

```javascript
{
  "result": {
    "error": "not_allowed"
  }
}
```

### auth\_on\_subscribe

Header: `vernemq-hook: auth_on_subscribe`

Webhook example payload:

```javascript
{
    "client_id": "clientid",
    "mountpoint": "",
    "username": "username",
    "topics":
        [{"topic": "a/b",
          "qos": 1},
         {"topic": "c/d",
          "qos": 2}]
}
```

A minimal response indicating the subscription authorization was successful looks like:

```javascript
{
    "result": "ok"
}
```

Another example where where the topics to subscribe have been rewritten looks like:

```javascript
{
    "result": "ok",
    "topics":
        [{"topic": "rewritten/topic",
          "qos": 0}]
}
```

Note, you can also pass a `qos` with value `128` which means it was either not possible or the client was not allowed to subscribe to that specific question.

Other possible responses:

```javascript
{
    "result": "next"
}
```

```javascript
{
    "result": { "error": "some error message" }
}
```

### auth\_on\_publish

Header: `vernemq-hook: auth_on_publish`

Note, in the example below the payload is not base64 encoded which is not the default.

Webhook example payload:

```javascript
{
    "username": "username",
    "client_id": "clientid",
    "mountpoint": "",
    "qos": 1,
    "topic": "a/b",
    "payload": "hello",
    "retain": false
}
```

A minimal response indicating the publish was authorized looks like:

```javascript
{
    "result": "ok"
}
```

A more complex example where the publish topic, qos, payload and retain flag is rewritten looks like:

```javascript
{
    "result": "ok",
    "modifiers": {
        "topic": "rewritten/topic",
        "qos": 2,
        "payload": "rewritten payload",
        "retain": true
    }
}
```

Other possible responses:

```javascript
{
    "result": "next"
}
```

```javascript
{
    "result": { "error": "some error message" }
}
```

### on\_register

Header: `vernemq-hook: on_register`

Webhook example payload:

```javascript
{
    "peer_addr": "127.0.0.1",
    "peer_port": 8888,
    "username": "username",
    "mountpoint": "",
    "client_id": "clientid"
}
```

The response should be an empty json object `{}`.

### on\_publish

Header: `vernemq-hook: on_publish`

Note, in the example below the payload is not base64 encoded which is not the default.

Webhook example payload:

```javascript
{
    "username": "username",
    "client_id": "clientid",
    "mountpoint": "",
    "qos": 1,
    "topic": "a/b",
    "payload": "hello",
    "retain": false
}
```

The response should be an empty json object `{}`.

### on\_subscribe

Header: `vernemq-hook: on_subscribe`

Webhook example payload:

```javascript
{
    "client_id": "clientid",
    "mountpoint": "",
    "username": "username",
    "topics":
        [{"topic": "a/b",
          "qos": 1},
         {"topic": "c/d",
          "qos": 2}]
}
```

The response should be an empty json object `{}`.

### on\_unsubscribe

Header: `vernemq-hook: on_unsubscribe`

Webhook example payload:

```javascript
{
    "username": "username",
    "client_id": "clientid",
    "mountpoint": "",
    "topics":
        ["a/b", "c/d"]
}
```

Example response:

```javascript
{
    "result": "ok",
    "topics":
        ["rewritten/topic"]
}
```

Other possible responses:

```javascript
{
    "result": "next"
}
```

```javascript
{
    "result": { "error": "some error message" }
}
```

### on\_deliver

Header: `vernemq-hook: on_deliver`

Note, in the example below the payload is not base64 encoded which is not the default.

Webhook example payload:

```javascript
{
    "username": "username",
    "client_id": "clientid",
    "mountpoint": "",
    "topic": "a/b",
    "payload": "hello"
}
```

Example response:

```javascript
{
  "result": "ok",
  "modifiers":
  {
        "topic": "rewritten/topic",
        "payload": "rewritten payload"
    }
}
```

Other possible responses:

```javascript
{
    "result": "next"
}
```

### on\_offline\_message

Header: `vernemq-hook: on_offline_message`

Note, in the example below the payload is not base64 encoded which is not the default.

Webhook example payload:

```javascript
{
    "client_id": "clientid",
    "mountpoint": "",
    "qos": "1",
    "topic": "sometopic",
    "payload": "payload",
    "retain": false
}
```

The response should be an empty json object `{}`.

### on\_client\_wakeup

Header: `vernemq-hook: on_client_wakeup`

Webhook example payload:

```javascript
{
    "client_id": "clientid",
    "mountpoint": ""
}
```

The response should be an empty json object `{}`.

### on\_client\_offline

Header: `vernemq-hook: on_client_offline`

Webhook example payload:

```javascript
{
    "client_id": "clientid",
    "mountpoint": ""
}
```

The response should be an empty json object `{}`.

### on\_client\_gone

Header: `vernemq-hook: on_client_gone`

Webhook example payload:

```javascript
{
    "client_id": "clientid",
    "mountpoint": ""
}
```

The response should be an empty json object `{}`.

### auth\_on\_register\_m5

Header: `vernemq-hook: auth_on_register_m5`

Webhook example payload:

```javascript
{
    "peer_addr": "127.0.0.1",
    "peer_port": 8888,
    "mountpoint": "",
    "client_id": "client-id",
    "username": "username",
    "password": "password",
    "clean_start": true,
    "properties": {}
}
```

A minimal response indicating the authentication was successful looks like:

```javascript
{
    "result": "ok"
}
```

It is also possible to override various client specific settings by returning an array of modifiers:

```javascript
{
    "result": "ok",
    "modifiers": {
        "max_message_size": 65535,
        "max_inflight_messages": 10000
    }
}
```

Note, the `retry_interval` is in milli-seconds. It is possible to override many more settings, see the [Session Lifecycle](sessionlifecycle.md) for more information.

Other possible responses:

```javascript
{
    "result": "next"
}
```

```javascript
{
    "result": {
      "error": "not_allowed"
  }
}
```

### on\_auth\_m5

Header `vernemq-hook: on_auth_m5`

Webhook example payload:

```javascript
{
    "username": "username",
    "mountpoint": "",
    "client_id": "client-id",
    "properties": {
      "p_authentication_data": "QVVUSF9EQVRBMA==",
      "p_authentication_method": "AUTH_METHOD"
    }
}
```

Note, as the authentication data is binary data it is base64 encoded.

A minimal response indicating the authentication was successful looks like:

```javascript
  "modifiers": {
    "properties": {
      "p_authentication_data": "QVVUSF9EQVRBMQ==",
      "p_authentication_method": "AUTH_METHOD"
    }
    "reason_code": 0
  },
  "result": "ok"
}
```

If authentication were to continue for another round a reason code with value 24 \(Continue Authentication\) should be returned instead. See also the relevant [section](http://docs.oasis-open.org/mqtt/mqtt/v5.0/cs02/mqtt-v5.0-cs02.html#_Toc514345528) in the MQTT 5.0 specification.

### auth\_on\_subscribe\_m5

Header: `vernemq-hook: auth_on_subscribe_m5`

Webhook example payload:

```javascript
{
    "username": "username",
    "mountpoint": "",
    "client_id": "client-id",
    "topics": [
      {
        "topic": "test/topic",
        "qos": 1
      }
    ],
    "properties": {}
  }
```

A minimal response indicating the subscription authorization was successful looks like:

```javascript
{
    "result": "ok"
}
```

Another example where where the topics to subscribe have been rewritten looks like:

```javascript
{
    "modifiers": {
        "topics": [
            {
                "qos": 2,
                "topic": "rewritten/topic"
            },
            {
                "qos": 135,
                "topic": "forbidden/topic"
            }
        ]
    },
    "result": "ok"
}
```

Note, the `forbidden/topic` has been rejected with the `qos` value of 135 \(Not authorized\).

Other responses

```text
{
    "result": "next"
}
```

```javascript
{
    "result": {
        "error": "not_allowed"
    }
}
```

### auth\_on\_publish\_m5

Header: `vernemq-hook: auth_on_publish_m5`

Note, in the example below the payload is not base64 encoded which is not the default.

Webhook example payload:

```javascript
{
    "username": "username",
    "mountpoint": "",
    "client_id": "client-id",
    "qos": 1,
    "topic": "some/topic",
    "payload": "message payload",
    "retain": false,
    "properties": {
    }
}
```

A minimal response indicating the publish was authorized looks like:

```javascript
{
    "result": "ok"
}
```

A response where the publish topic has been rewritten:

```javascript
{
    "modifiers": {
        "topic": "rewritten/topic"
    },
    "result": "ok"
}
```

Other possible responses:

```javascript
{
    "result": "next"
}
```

```javascript
{
    "result": {
        "error": "not_allowed"
    }
}
```

### on\_register\_m5

Header: `vernemq-hook: on_register_m5`

Webhook example payload:

```javascript
{
    "peer_addr": "127.0.0.1",
    "peer_port": 8888,
    "mountpoint": "",
    "client_id": "client-id",
    "username": "username",
    "properties": {
    }
}
```

The response should be an empty json object `{}`.

### on\_publish\_m5

Header: `vernemq-hook: on_publish_m5`

Note, in the example below the payload is base64 encoded .

Webhook example payload:

```javascript
{
    "username": "username",
    "mountpoint": "",
    "client_id": "client-id",
    "qos": 1,
    "topic": "test/topic",
    "payload": "message payload",
    "retain": false,
    "properties": {
    }
}
```

The response should be an empty json object `{}`.

### on\_subscribe\_m5

Header: `vernemq-hook: on_subscribe_m5`

Webhook example payload:

```javascript
{
    "username": "username",
    "mountpoint": "",
    "client_id": "client-id",
    "topics": [
        {
            "topic": "test/topic",
            "qos": 1
        },
        {
            "topic": "test/topic",
            "qos": 128
        }
    ],
    "properties": {
    }
}
```

Note, the qos value of `128` \(Unspecified error\) means the subscription was rejected.

The response should be an empty json object `{}`.

### on\_unsubscribe\_m5

Header: `vernemq-hook: on_unsubscribe_m5`

Webhook example payload:

```javascript
{
    "username": "username",
    "mountpoint": "",
    "client_id": "client-id",
    "topics": [
        "test/topic"
    ],
    "properties": {
    }
}
```

Example response:

```javascript
{
    "modifiers": {
        "topics": [
            "rewritten/topic"
        ]
    },
    "result": "ok"
}
```

Other possible responses:

```javascript
{
    "result": "next"
}
```

### on\_deliver\_m5

Header: `vernemq-hook: on_deliver_m5`

Note, in the example below the payload is not base64 encoded which is not the default.

Webhook example payload:

```javascript
{
    "username": "username",
    "mountpoint": "",
    "client_id": "client-id",
    "topic": "test/topic",
    "payload": "message payload",
    "properties": {
    }
}
```

Example response:

```javascript
{
    "result": "ok"
}
```

Other possible responses:

```javascript
{
    "result": "next"
}
```

## Example Webhook in Python

Below is a very simple example of an endpoint implemented in Python. It uses the `web` and `json` modules and implements handlers for three different hooks: `auth_on_register`, `auth_on_publish` and `auth_on_subscribe`.

The `auth_on_register` hook only restricts access only to the user with username `joe` and password `secret`. The `auth_on_subscribe` and `auth_on_publish` hooks allow any subscription or publish to continue as is. These last two hooks are needed as the default policy is `deny`.

```python
import web
import json

urls = ('/.*', 'hooks')
app = web.application(urls, globals())

class hooks:
    def POST(self):

        # fetch hook and request data
        hook = web.ctx.env.get('HTTP_VERNEMQ_HOOK')
        data = json.loads(web.data())

        # print the hook and request data to the console
        print
        print 'hook:', hook
        print '  data: ', data

        # dispatch to appropriate function based on the hook.
        if hook == 'auth_on_register':
            return handle_auth_on_register(data)
        elif hook == 'auth_on_publish':
            return handle_auth_on_publish(data)
        elif hook == 'auth_on_subscribe':
            return handle_auth_on_subscribe(data)
        else:
            web.ctx.status = 501
            return "not implemented"

def handle_auth_on_register(data):
    # only allow user 'joe' with password 'secret', reject all others.
    if "joe" == data['username']:
        if "secret" == data['password']:
            return json.dumps({'result': 'ok'})

    return json.dumps({'result': {'error': 'not allowed'}})

def handle_auth_on_publish(data):
    # accept all publish requests
    return json.dumps({'result': 'ok'})

def handle_auth_on_subscribe(data):
    # accept all subscribe requests
    return json.dumps({'result': 'ok'})

if __name__ == '__main__':
    app.run()
```

