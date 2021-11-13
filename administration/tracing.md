---
description: Real-time inspection
---

# Tracing

## Introduction

When working with a system like VerneMQ sometimes when troubleshooting it would be nice to know what a client is actually sending and receiving and what VerneMQ is doing with this information. For this purpose VerneMQ has a built-in tracing mechanism which is safe to use in production settings as there is very little overhead in running the tracer and has built-in protection mechanisms to stop traces that produce too much information.

## Tracing clients

To trace a client the following command is available:

```text
vmq-admin trace client client-id=<client-id>
```

See the available flags by calling `vmq-admin trace client --help`.

A typical trace could look like the following:

```text
$ vmq-admin trace client client-id=client
No sessions found for client "client"
New session with PID <7616.3443.1> found for client "client"
<7616.3443.1> MQTT RECV: CID: "client" CONNECT(c: client, v: 4, u: username, p: password, cs: 1, ka: 30)
<7616.3443.1> Calling auth_on_register({{172,17,0,1},34274},{[],<<"client">>},username,password,true) 
<7616.3443.1> Hook returned "ok"
<7616.3443.1> MQTT SEND: CID: "client" CONNACK(sp: 0, rc: 0)
<7616.3443.1> MQTT RECV: CID: "client" SUBSCRIBE(m1) with topics:
    q:0, t: "topic"
<7616.3443.1> Calling auth_on_subscribe(username,{[],<<"client">>}) with topics:
    q:0, t: "topic"
<7616.3443.1> Hook returned "ok"
<7616.3443.1> MQTT SEND: CID: "client" SUBACK(m1, qt[0])
<7616.3443.1> Trace session for client stopped
```

In this particular trace a trace was started for the client with client-id `client`. At first no clients are connected to the node where the trace has been started, but a little later the client connects and we see the trace come alive. The strange identifier `<7616.3443.1>` is called a process identifier and is the identifier of the process in which the trace happened - this isn't relevant unless one wants to correlate the trace with log entries where process identifiers are also logged. Besides the process identifier there are some lines with `MQTT SEND` and `MQTT RECV` which are to be understood from the perspective of the broker. In the above trace this means that first the broker receives a `CONNECT` frame and replies with a `CONNACK` frame. Each MQTT event is annotated with the data from the MQTT frame to give as much detail and insight as possible.

Notice the `auth_on_register` call between `CONNECT` and `CONNACK` which is the authentication plugin hook being called to authenticate the client. In this case the hook returned `ok` which means the client was successfully authenticated.

Likewise notice the `auth_on_subscribe` call between the `SUBSCRIBE` and `SUBACK` frames which is plugin hook used to authorize if this particular subscription should be allowed or not. In this case the subscription was authorized.

### Trace options

The client trace command has additional options as shown by `vmq-admin trace client --help`. Those are hopefully self-explaining:

```text
Options

  --mountpoint=<Mountpoint>
      the mountpoint for the client to trace.
      Defaults to "" which is the default mountpoint.
  --rate-max=<RateMaxMessages>
      the maximum number of messages for the given interval,
      defaults to 10.
  --rate-interval=<RateIntervalMS>
      the interval in milliseconds over which the max number of messages
      is allowed. Defaults to 100.
  --trunc-payload=<SizeInBytes>
      control when the payload should be truncated for display.
      Defaults to 200.
```

{% hint style="info" %}
A convenient tool is the `ts` \(timestamp\) tool which is available on many systems. If the trace output is piped to this command each line is prefixed with a timestamp:

`ts | sudo vmq-admin trace client client-id=tester`
{% endhint %}

{% hint style="info" %}
It is currently not possible to start multiple traces from multiple shells, or trace multiple ClientIDs.
{% endhint %}

