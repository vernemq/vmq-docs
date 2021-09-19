# Introduction

On every VerneMQ node you'll find the `vmq-admin` command line tool in the release's bin directory. It has different sub-commands that let you check for status, start and stop listeners, re-configure values and a couple of other administrative tasks.

{% hint style="info" %}
`vmq-admin` works by RPC'ing into the local VerneMQ node by default. For most commands you can add a `--node` option and set values on other cluster nodes, even if the local VerneMQ node is down.   
  
To check for the global cluster state in case the local VerneMQ node is down, you'll have to go to another node though.
{% endhint %}

{% hint style="danger" %}
`vmq-admin` is a live re-configuration utility. Please note that all dynamically configured values will be reset by vernemq.conf upon broker restart.   
 Don't use this to wildly re-configure a production system without keeping track what you are doing. `vmq-admin` gives you the flexibility to test stuff and react live, but please persistent any static configuration you need in the vernemq.conf file.
{% endhint %}

