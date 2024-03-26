---
description: Managing VerneMQ tcp listeners
---

# Managing Listeners

You can configure as many listeners as you wish in the vernemq.conf file. In addition to this, the `vmq-admin listener` command let's you configure, start, stop and delete listeners on the fly. Those can be MQTT, WebSocket or Cluster listeners, in the command line output they will be tagged mqtt, ws or vmq accordingly.

{% hint style="info" %}
To get info on a listener sub-command, invoke it with the --help option. Example: `vmq-admin listener start --help`
{% endhint %}

{% hint style="warning" %}
Listeners configured with the `vmq-admin listener` command will not survive a broker restart. Live changes to listeners configured in vernemq.conf are possible, but the vernemq.conf listeners will just be restarted with a broker restart.
{% endhint %}

## Status of all listeners

```text
vmq-admin listener show
```

```text
    +----+-------+------------+-----+----------+---------+
    |type|status |     ip     |port |mountpoint|max_conns|
    +----+-------+------------+-----+----------+---------+
    |vmq |running|192.168.1.50|44053|          |  30000  |
    |mqtt|running|192.168.1.50|1883 |          |  30000  |
    +----+-------+------------+-----+----------+---------+
`
```

You can retrieve additional information by adding the --tls or --mqtt switch. See 

```text
vmq-admin listener show --help 
```

for more information.

## Starting a new listener

```text
vmq-admin listener start address=192.168.1.50 port=1884 --mountpoint /test --nr_of_acceptors=10 --max_connections=1000
```

This will start an MQTT listener on port `1884` and IP address `192.168.1.50`. If you want to start a WebSocket listener, just tell VerneMQ by adding the `--websocket` flag. There are more options, mainly for configuring SSL \(use `vmq-admin listener start --help`\).

You can isolate client connections accepted by a certain listener from other clients by setting a mountpoint.

To start an MQTT listener using defaults, just set the port and IP address as a minimum.

## Stopping a listener

```text
vmq-admin listener stop address=192.168.1.50 port=1884
```

A stopped listener will not accept new connections, but continue existing sessions. You can add the `-k` or `--kill_sessions` switch to that command. This will disconnect all client connections setup by that listener. In combination with a mountpoint, this can be useful for terminating clients for a specific application, or to force re-connects to another cluster node \(to prepare for a cluster leave for your node\).

## Restarting a stopped listener

```text
vmq-admin listener restart address=192.168.1.50 port=1884
```

## Deleting a stopped listener

```text
vmq-admin listener delete address=192.168.1.50 port=1884
```

