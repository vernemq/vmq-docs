---
description: Configure a couple of hidden options for VerneMQ
---

# Advanced Options

There are a couple of hidden options you can set in the `vernemq.conf` file. Hidden means that you have to add and set the value explicitly. Hidden options still have default values. Changing them should be considered advanced, possibly with the exception of setting a `max_message_rate`.

## Queue Deliver mode

Specify how the queue should deliver messages when multiple sessions are allowed. In case of `fanout` all the attached sessions will receive the message, in case of `balance` an attached session is choosen randomly.

{% hint style="info" %} The feature to enable multiple sessions will be deprecated in VerneMQ 2.0.{% endhint %}

```text
queue_deliver_mode = balance
```

## Queue Type

Specify how queues should process messages, either the `fifo` or `lifo` way, with a default setting of `fifo`. The setting will apply globally, that is, for every spawned queue in a VerneMQ broker. (You can override the `queue_type` setting in plugins in the `auth_on_register` hook).

```text
queue_type = fifo
```

## Max Message Rate

Specifies the maximum incoming publish rate per session per second. Depending on the underlying network buffers this rate isn't enforced. Defaults to `0`, which means no rate limits apply. Setting to a value of `2` limits any publisher to 2 messages per second, for instance.

```text
max_message_rate = 2
```

## Max Drain Time

Due to the eventually consistent nature of the subscriber store it is possible that during queue migration messages still arrive on the old cluster node. This parameter enables compensation for that fact by keeping the queue around for some configured time \(in seconds\) after it was migrated to the other cluster node.

```text
max_drain_time = 20
```

## Max Msgs per Drain Step

Specifies the number of messages that are delivered to the remote node per drain step. A large value will provide a faster migration of a queue, but increases the waste of bandwidth in case the migration fails.

```text
max_msgs_per_drain_step = 1000
```

## Default Reg View

Allows to select a new default reg\_view. A reg\_view is a pre-defined way to route messages. Multiple views can be loaded and used, but one has to be selected as a default. The default routing is `vmq_reg_trie`, i.e. routing via the built-in trie data structure.

```text
vmq_reg_view = "vmq_reg_trie"
```

## Reg Views

A list of views that are started during startup. It's only used in plugins that want to choose dynamically between routing reg\_views.

```text
reg_views = "[vmq_reg_trie]"
```

## Outgoing Clustering Buffer Size

An integer specifying how many bytes are buffered in case the remote node is not available. Default is `10000`

```text
outgoing_clustering_buffer_size = 15000
```

## Max Connection Lifetime
Defines the maximum lifetime of MQTT connection in seconds. Max_connection_lifetime can be set per-listener.  This is an implementation of MQTT security proposal:
"Servers may close the Network Connection of Clients and require them to re-authenticate with new credentials."

```text
listener.max_connection_lifetime = 25000
```

It is possible to override the value in auth_on_register(_m5) to a lower limit.

