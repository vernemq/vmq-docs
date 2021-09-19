---
description: This describes a quick way to create a VerneMQ cluster on developer's machines
---

# Clustering during development

Sometimes you want to have a quick way to test a cluster on your development machine as a VerneMQ developer. 

You need to take care of a couple things if you want to run multiple VerneMQ instances on the same machine. There is a `make` option that let's you build multiple releases, as a commodity, taking care of all the configuration. 

First, build a normal release \(this is just needed the first time\) with:

`➜ default git:(master) ✗ make rel`

The following command will then prepare 3 correctly configured vernemq.conf files, with different ports for the MQTT listeners etc. It will also build 3 full VerneMQ releases.

 `➜ default git:(master) ✗ make dev1 dev2 dev3`

Check if you have the 3 new releases in the `_build` directory of your VerneMQ code repo. 

You can then start the respective broker instances in 3 terminal windows, by using the respective commands and directory paths. Example:

`➜ (_build/dev2/rel/vernemq/bin) ✗ vernemq console`

The MQTT listeners will of course be configured differently for each node \(the default 1883 port is not used, so that you can still run a default MQTT broker besides your dev nodes\). A couple of other ports are also adapted \(HTTP status page, cluster communication\). The MQTT ports are automically configured in increasing steps of 50: \(if in doubt, consult the respective `vernemq.conf` files\)

| Node | MQTT listener port |
| :--- | :--- |
| dev1@127.0.0.1 | 10053 |
| dev2@127.0.0.1 | 10103 |
| dev3@127.0.0.1 | 10153 |
| ... | ... |

Note that the dev nodes are not automatically clustered. You still need to manually cluster them with commands like the following:

`➜ (_build/dev2/rel/vernemq/bin) ✗ vmq-admin cluster join discovery-node=dev1@127.0.0.1`

{% hint style="info" %}
In case this wasn't clear so far: You can configure an arbitrary number of cluster nodes, from dev1 to devn.
{% endhint %}

