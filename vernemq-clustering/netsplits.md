---
description: How does VerneMQ deals with Network Partitions aka. Netsplits.
---

# Dealing with Netsplits

This section elaborates how a VerneMQ cluster deals with network partitions \(aka. netsplit or split brain situation\). A netsplit is mostly the result of a failure of one or more network devices resulting in a cluster where nodes can no longer reach each other.

VerneMQ is able to detect a network partition, and by default it will stop serving `CONNECT`, `PUBLISH`, `SUBSCRIBE`, and `UNSUBSCRIBE` requests. A properly implemented client will always resend unacked commands and messages are therefore not lost \(QoS 0 publishes will be lost\). However, the time window between the network partition and the time VerneMQ detects the partition **much** can happen. Moreover, this time frame will be different on every participating cluster node. In this guide we're referring to this time frame as the _Window of Uncertainty_.

{% hint style="success" %}
The behaviour during a netsplit is completely configurable via `allow_register_during_netsplit`, `allow_publish_during_netsplit`, `allow_subscribe_during_netsplit`, and `allow_unsubscribe_during_netsplit`. These options supersede the `trade_consistency` option. In order to reach the same behaviour as `trade_consistency = on` all the mentioned netsplit options have to set to `on`.
{% endhint %}

## Possible Scenario for Message Loss:

VerneMQ follows an eventually consistent model for storing and replicating the subscription data. This also includes retained messages.

Due to the eventually consistent data model it is possible that during the Window of Uncertainty a publish won't take into account a subscription made on a remote node \(in another partition\). Obviously, VerneMQ can't deliver the message in this case. The same holds for delivering retained messages to remote subscribers.

`last will` messages that are triggered during the Window of Uncertainty will be delivered to the reachable subscribers. Currently during a netsplit, but after the Window of Uncertainty last will messages will be lost.

## Possible Scenario for Duplicate Clients:

Normally, client registration is synchronized using an _elected_ leader node for the given client id. Such a synchronization removes the race condition between multiple clients trying to connect with the same client id on different nodes. However, during the Window of Uncertainty it is currently possible that VerneMQ fails to disconnect a client connected to a different node. Although this scenario sounds like artificially crafted it is possible to end up with duplicate clients connected to the cluster.

## Recovering from a Netsplit

As soon as the partition is healed, and connectivity reestablished, the VerneMQ nodes replicate the latest changes made to the subscription data. This includes all the changes 'accidentally' made during the Window of Uncertainty. Using [Dotted Version Vectors](https://github.com/ricardobcl/Dotted-Version-Vectors) VerneMQ ensures that convergence regarding subscription data and retained messages is eventually reached.

