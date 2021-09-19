---
description: Managing VerneMQ live config values.
---

# Live reconfiguration

You can dynamically re-configure most of VerneMQ's settings on a running node by using the `vmq-admin set` command.

The following config values can be handled dynamically:

```text
allow_anonymous
topic_alias_max_broker
receive_max_broker
vmq_acl.acl_file
graphite_host
vmq_acl.acl_reload_interval
graphite_enabled
queue_type
suppress_lwt_on_session_takeover
max_message_size
vmq_passwd.password_file
graphite_port
max_client_id_size
upgrade_outgoing_qos
max_message_rate
graphite_interval
allow_multiple_sessions
systree_enabled
max_last_will_delay
retry_interval
receive_max_client
max_offline_messages
max_online_messages
max_inflight_messages
allow_register_during_netsplit
vmq_passwd.password_reload_interval
topic_alias_max_client
systree_interval
allow_publish_during_netsplit
coordinate_registrations
remote_enqueue_timeout
persistent_client_expiration
allow_unsubscribe_during_netsplit
graphite_include_labels
shared_subscription_policy
queue_deliver_mode
allow_subscribe_during_netsplit
```

{% hint style="warning" %}
Settings dynamically configured with the `vmq-admin set` command will be reset by vernemq.conf upon broker restart.
{% endhint %}

## Setting a value for the local node

Let's change the `max_client_id_size` as an example. \(We might have noticed that some clients can't login because their client ID is too long, but we don't want to restart the broker for that\). Note that you can also set multiple values with the same command.

```text
vmq-admin set max_client_id_size=45
```

## Setting a value for an arbitrary cluster node

```text
vmq-admin set max_client_id_size=45 --node=VerneMQ20@192.168.1.20
```

## Setting a value for all cluster nodes

```text
vmq-admin set max_client_id_size=45 --all
```

## Show current VerneMQ config values

### For the local node

You can show one or multiple values in a simple table:

```text
vmq-admin show max_client_id_size retry_interval
```

```text
+----------------------+------------------+--------------+
|         node         |max_client_id_size|retry_interval|
+----------------------+------------------+--------------+
|VerneMQ20@192.168.1.50|        28        |      20      |
+----------------------+------------------+--------------+

`
```

### For an arbitrary node

```text
vmq-admin show max_client_id_size retry_interval --node VerneMQ20@192.168.1.20
```

### For all cluster nodes

```text
vmq-admin show max_client_id_size retry_interval --all
```

```text
+----------------------+------------------+--------------+
|         node         |max_client_id_size|retry_interval|
+----------------------+------------------+--------------+
|VerneMQ30@192.168.1.30|        33        |      20      |
|VerneMQ40@192.168.1.40|        33        |      20      |
|VerneMQ10@192.168.1.10|        33        |      20      |
|VerneMQ50@192.168.1.50|        33        |      20      |
|VerneMQ20@192.168.1.20|        28        |      20      |
+----------------------+------------------+--------------+
```

