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

**Attributions:**

This section, "VerneMQ Inter-node Communication", is a derivative of Security and Firewalls by Riak, used under Creative Commons Attribution 3.0 Unported License.

