# Introduction

On every VerneMQ node you'll find the `vmq-admin` command line tool in the release's bin directory (in case you use the binary VerneMQ packages, `vmq-admin` should already be callable in your path, without changing directories). It has different sub-commands that let you check for status, start and stop listeners, re-configure values and a couple of other administrative tasks.

`vmq-admin` has different sub-commands with a lot of respective options. You can familiarize yourself by using the `--help` option on the different levels of `vmq-admin`. You might see additional sub-commands in case integrated plugins are running (`vmq-admin bridge` is an example).

```text
$ sudo vmq-admin --help       
Usage: vmq-admin <sub-command>

  Administrate the cluster.

  Sub-commands:
    node        Manage this node
    cluster     Manage this node's cluster membership
    session     Retrieve session information
    retain      Show and filter MQTT retained messages
    plugin      Manage plugin system
    listener    Manage listener interfaces
    metrics     Retrieve System Metrics
    api-key     Manage API keys for the HTTP management interface
    trace       Trace various aspects of VerneMQ
  Use --help after a sub-command for more details.
```

{% hint style="info" %}
`vmq-admin` works by RPC'ing into the local VerneMQ node by default. For most commands you can add a `--node` option and set values on other cluster nodes, even if the local VerneMQ node is down.   
  
To check for the global cluster state in case the local VerneMQ node is down, you'll have to go to another node though.
{% endhint %}

{% hint style="danger" %}
`vmq-admin` is a live re-configuration utility. Please note that all dynamically configured values will be reset by vernemq.conf upon broker restart.   
 As a consequence, it's good practice to keep track of the applied changes when re-configuring a broker with `vmq-admin`. If needed, you can then persist changes by adding them to the vernemq.conf file.
{% endhint %}

