---
description: MQTT consumers can share and loadbalance a topic subscription.
---

# Consumer session balancing

{% hint style="warning" %}
Consumer session balancing has been deprecated and will be removed in VerneMQ 2.0. Use [Shared Subscriptions](https://github.com/vernemq/vmq-docs/tree/d773fff8c46594ee0cfd3b278c3c12273c13a630/configuration/configuration/shared_subscriptions/README.md) instead.
{% endhint %}

Sometimes consumers get overwhelmed by the number of messages they receive. VerneMQ can load balance between multiple consumer instances subscribed to the same topic with the same ClientId.

## Enabling Session Balancing

To enable session balancing, activate the following two settings in vernemq.conf

```text
 allow_multiple_sessions = on
 queue_deliver_mode = balance
```

{% hint style="info" %}
Currently those settings will activate consumer session balancing globally on the respective node. Restricting balancing to specific consumers only, will require a plugin. Note that you cannot balance consumers spread over different cluster nodes.
{% endhint %}

