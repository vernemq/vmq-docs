---
description: Everything you must know to properly configure and deploy a VerneMQ Cluster
---

# Inter-node Communication

VerneMQ uses the Erlang distribution mechanism for most inter-node communication. VerneMQ identifies other machines in the cluster using Erlang identifiers \(e.g. `VerneMQ@10.9.8.7`\). Erlang resolves these node identifiers to a TCP port on a given machine via the Erlang Port Mapper daemon \(epmd\) running on each cluster node.

By default, epmd binds to TCP port 4369 and listens on the wildcard interface. For inter-node communication, Erlang uses an unpredictable port by default; it binds to port 0, which means the first available port.

For ease of firewall configuration, VerneMQ can be configured to instruct the Erlang interpreter to use a limited range of ports. For example, to restrict the range of ports that Erlang will use for inter-Erlang node communication to 6000-7999, add the following lines to vernemq.conf on each VerneMQ node:

```text
erlang.distribution.port_range.minimum = 6000
erlang.distribution.port_range.maximum = 7999
```

The settings above are only used for distributing subscription updates and maintenance messages. For distributing the 'real' MQTT messages the proper `vmq` listener must be configured in the vernemq.conf.

```text
listener.vmq.clustering = 0.0.0.0:44053
```

{% hint style="info" %}
It isn't necessary to configure the same port on every machine, as the nodes will probe each other for this information.
{% endhint %}

## Internode MQTT Delivery

VerneMQ uses MQTT connections between cluster nodes to deliver MQTT messages to subscribers on remote nodes. In most deployments, the defaults should be left unchanged. The following settings are advanced options for deployments that need to tune behavior around slow peers, short disconnects, reconnect handshakes, or unusually large bursts of internode traffic.

`outgoing_cluster_handshake_ack_timeout` configures how long a node waits for the receiving side to acknowledge an outgoing cluster delivery connection before assuming legacy behavior. The value is in milliseconds and defaults to `250`.

```text
outgoing_cluster_handshake_ack_timeout = 250
```

`incoming_clustering_buffer_size` limits how many bytes are buffered while parsing incoming internode MQTT traffic. Malformed or oversized cluster frames are rejected instead of being buffered indefinitely. The value is in bytes and defaults to `67108864`.

```text
incoming_clustering_buffer_size = 67108864
```

`outgoing_clustering_buffer_size` configures how many bytes are buffered when a remote node is temporarily unavailable. The value is in bytes and defaults to `67108864`.

```text
outgoing_clustering_buffer_size = 67108864
```

`outgoing_clustering_flush_threshold` configures how many pending outgoing bytes are batched before VerneMQ eagerly flushes internode traffic. The value is in bytes and defaults to `65536`.

```text
outgoing_clustering_flush_threshold = 65536
```

Change these values only after testing with realistic cluster traffic. Setting buffers too low can increase dropped internode traffic during transient disconnects, while setting them too high can increase memory usage under sustained peer or network problems.

**Attributions:**

This section, "VerneMQ Inter-node Communication", is a derivative of Security and Firewalls by Riak, used under Creative Commons Attribution 3.0 Unported License.
