---
description: Inspecting and managing MQTT sessions
---

# Inspecting and managing sessions

## Inspecting sessions

VerneMQ comes with powerful tools for inspecting the state of MQTT sessions. To list current MQTT sessions simply invoke `vmq-admin session show`:

```text
$ vmq-admin session show
+---------+---------+----------+---------+---------+---------+
|client_id|is_online|mountpoint|peer_host|peer_port|  user   |
+---------+---------+----------+---------+---------+---------+
| client2 |  true   |          |127.0.0.1|  37098  |undefined|
| client1 |  true   |          |127.0.0.1|  37094  |undefined|
+---------+---------+----------+---------+---------+---------+
```

To see detailed information about the command see `vmq-admin session show --help`.

The command is able to show a lot of different information about a client, for example the client id, the peer host and port if the client is online or offline and much more, see `vmq-admin session show --help` for details. Further the information can also be used to filter information which is very helpful when wanting to narrow down the information to a single client.

A sample query which lists only the node where the client session exists and if the client is online would look like the following:

```text
$ vmq-admin session show --node --is_online --client_id=client1
+---------+--------------+
|is_online|     node     |
+---------+--------------+
|  true   |dev2@127.0.0.1|
+---------+--------------+
```

{% hint style="success" %}
Note, by default a maximum of 100 rows are returned from each node in the cluster. This is a mechanism to protect the cluster from overload as there can be millions of MQTT sessions and resulting rows. Use `--limit=<RowLimit>` to override the default value.
{% endhint %}

### More examples

Listing the clients and the subscriptions one can do the following:

```text
$ vmq-admin session show --topic --client_id
+---------+-----------------+
|client_id|     topic       |
+---------+-----------------+
| client2 |some/other/topic1|
| client1 |some/other/topic2|
| client1 |   some/topic    |
+---------+-----------------+
```

And to list only the clients subscribed to the topic `some/topic`:

```text
$ vmq-admin session show --topic --client_id --topic=some/topic
+---------+----------+
|client_id|  topic   |
+---------+----------+
| client1 |some/topic|
+---------+----------+
```

you can also do a regex search to query a subset of topics:

```text
$ vmq-admin session show --topic --client_id --topic=~some/other/.*
+---------+-----------------+
|client_id|      topic      |
+---------+-----------------+
| client2 |some/other/topic1|
| client1 |some/other/topic |
+---------+-----------------+
```

A regex search uses the =~ syntax and is currently limited to alpha-numeric searches. Please note that the regex search consumes more load an a node than a regular search.

```text
$ vmq-admin session show --topic --client_id --topic=some/topic
+---------+----------+
|client_id|  topic   |
+---------+----------+
| client1 |some/topic|
+---------+----------+
```

To figure out when the queue for a persisted session \(clean\_session=false\) was created and when the client last connected one can use the `--queue_started_at` and `--session_started_at` to list the POSIX timestamps \(in microseconds\):

```text
$ vmq-admin session show --client_id=client1 --queue_started_at --session_started_at
+----------------+------------------+
|queue_started_at|session_started_at|
+----------------+------------------+
| 1549379963575  |  1549379974905   |
+----------------+------------------+
```

Besides the examples above it is also possible to inspect the number of online or offline messages as well as their payloads and much more. See `vmq-admin session show --help` for an exhaustive list of all the available options.

## Managing sessions

VerneMQ also supports disconnecting clients and reauthorizing client subscriptions. To disconnect a client and cleanup store messages and remove subscriptions one can invoke:

```text
$ vmq-admin session disconnect client-id=client1 --cleanup
```

See `vmq-admin session disconnect --help` for more options and details.

To reauthorize subscriptions for a client issue the following command:

```text
$ vmq-admin session reauthorize username=username client-id=client1
Unchanged
```

This works by reapplying the logic in any installed `auth_on_subscribe` or `auth_on_subscribe_m5` plugin hooks to check the validity of the existing topics and removing those that are no longer allowed. In the example above the reauthorization of the client subscriptions resulted in no changes.

