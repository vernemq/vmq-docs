---
description: Everything you must know to properly configure and deploy a VerneMQ Cluster
---

# Introduction

VerneMQ can be easily clustered. Clients can then connect to any cluster node and receive messages from any other cluster nodes. However, the MQTT specification gives certain guarantees that are hard to fulfill in a distributed environment, especially when network partitions occur. We'll discuss the way VerneMQ deals with network partitions in its [own subsection](netsplits.md)

{% hint style="danger" %}
**Set the Cookie!** All cluster nodes need to be configured to use the same Cookie value. It can be set in the `vernemq.conf` with the `distributed_cookie` setting. Set the Cookie to a private value for security reasons!
{% endhint %}

{% hint style="info" %}
For a successful VerneMQ cluster setup, it is important to choose proper VerneMQ node names. In `vernemq.conf` change the `nodename = VerneMQ@127.0.0.1` to something appropriate. Make sure that the node names are unique within the cluster. Read the section on [VerneMQ Inter-node Communication](communication.md) if firewalls are involved.
{% endhint %}

#### A note on statefulness

Before you go ahead and experience the full power of clustering VerneMQ, be aware of its stateful character. An MQTT broker is a stateful application and a VerneMQ cluster is a stateful cluster.

What does this mean in detail? It means that clustered VerneMQ nodes will share information about connected clients and sessions but also meta-information about the cluster itself.

For instance, if you stop a cluster node, the VerneMQ cluster will not just forget about it. It will know that there's a node missing and it will keep looking for it. It will know there's a netsplit situation and it will heal the partition when the node comes back up. But if the missing node never comes back there's an eternal netsplit. \(still resolvable by making the missing node explicitly leave\).

This doesn't mean that a VerneMQ cluster cannot dynamically grow and shrink. But it means you have to tell the cluster what you intend to do, by using join and leave commands.

If you want a cluster node to leave the cluster, well... use the `vmq-admin cluster leave` command. If you want a node to join a cluster use the `vmq-admin cluster join` command.

Makes sense? Go ahead and create your first VerneMQ cluster!

## Joining a Cluster

```text
vmq-admin cluster join discovery-node=<OtherClusterNode>
```
The discovery-node can be any other node. It is not necessary to always choose the same node as discovery node. It is important that only a node with an empty history joins a cluster. One should not try to add a node, that had already traffic on it, to a cluster.   

## Leaving a Cluster

```text
vmq-admin cluster leave node=<NodeThatShouldGo> (only the first step!)
```

## Detailed Cluster Leave, Case A: Make a live node leave

A cluster leave will actually do a lot more work, and gives you some options to choose. The node leaving the cluster will go to great length trying to migrate its existing queues to other nodes. As queues \(online or offline\) are live processes in a VerneMQ node, it will only exit after it has migrated them.

Let's look at the steps in detail:

1. `vmq-admin cluster leave node=<NodeThatShouldGo>`

This first step will only **stop** the MQTT Listeners of the node to ensure that no new connections are accepted. It will **not** interrupt the existing connections, and behind the scenes the node will **not** leave the cluster yet. Existing clients are still able to publish and receive messages at this point.

The idea is to give a grace period with the hope that existing clients might re-connect \(to another node\). If you have decided that this period is over \(after 5 minutes or 1 day is up to you\), you proceed with step 2: disconnecting the rest of the clients.

1. `vmq-admin cluster leave node=<NodeThatShouldGo> -k`

The `-k` flag will **delete** the MQTT Listeners of the leaving node, taking down all live connections. If this is what you want from the beginning, you can do this right away as a first step.

Now, queue migration is triggered by clients re-connecting to other nodes. They will claim their queue and it will get migrated. Still, there might be some offline queues remaining on the leaving node, because they were pre-existing or because some clients do not re-connect and do not reclaim their queues.

VerneMQ will throw an exception if there are remaining offline queues after a configurable timeout. The default is 60 seconds, but you can set it as an option to the cluster leave command. As soon as the exception shows in console or console.log, you can actually retry the cluster leave command \(including setting a migration timeout \(`-t`\), and an interval in seconds \(`-i`\) indicating how often information on the migration progress should be printed to the console.log\):

1. `vmq-admin cluster leave node=<NodeThatShouldGo> -k -i 5 -t 120`

After this timeout VerneMQ will forcefully migrate the remaining offline queues to other cluster nodes in a round robin manner. After doing that, it will stop the leaving VerneMQ node.

{% hint style="info" %}
**Note 1:** While doing a cluster leave, it's a good idea to tail -f the VerneMQ console.log to see queue migration progress.
{% endhint %}

{% hint style="info" %}
**Note 2:** A node that has left the cluster is considered dead. If you want to reuse that node as a single node broker, you have to \(backup & rename &\) delete the whole VerneMQ`data` directory and start with a new directory. \(It will be created automatically by VerneMQ at boot\).

Otherwise that node will start looking for its old cluster peers when you restart it.
{% endhint %}

## Detailed Cluster Leave, Case B: Make a stopped node leave

So, case A was the happy case. You left the cluster with your node in a controlled manner, and everything worked, including a complete queue \(and message\) transfer to other nodes.

Let's look at the second possibility where the node is already down. Your cluster is still counting on it though and possibly blocking new subscription for that reason, so you want to make the node leave.

To do this, use the same command\(s\) as in the first case. There is one important consequence to note: by making a stopped node leave, you basically throw away persistant queue content, as VerneMQ won't be able to migrate or deliver it.

Let's repeat that to make sure:

{% hint style="danger" %}
**Case B:** Currently the persisted QoS 1 & QoS 2 messages aren't replicated to the other nodes by the default message store backend. Currently you will **lose** the offline messages stored on the leaving node.
{% endhint %}

## Getting Cluster Status Information

```text
vmq-admin cluster show
```

