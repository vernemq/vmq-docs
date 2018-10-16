---
description: Description and Configuration of the $SYSTree Monitoring Feature
---

# $SYSTree

The systree functionality is enabled by default and reports the broker metrics at a fixed interval defined in the `vernemq.conf`. The metrics defined [here](introduction.md) are transformed to MQTT topics e.g. `mqtt_publish_received` is transformed to `$SYS/<nodename>/mqtt/publish/received`. `<nodename>` is your node's name, as configured in the `vernemq.conf`. To find it, you can grep the file for it: `grep nodename vernemq.conf`

The complete list of metrics can be found [here.](introduction.md)

```text
systree_interval = 20000
```

This option defaults to `20000` milliseconds.

If the systree feature is not required it can be disabled in `vernemq.conf`

```text
systree_enabled = off
```

{% hint style="success" %}
The feature and the interval can be changed at runtime using the `vmq-admin` script.  
 Usage: vmq-admin set = ... \[\[--node \| -n\]  \| --all\]   
 Example: `vmq-admin set systree_interval=60000 -n VerneMQ@127.0.0.1`
{% endhint %}

Examples:

```text
mosquitto_sub -t \$SYS/<node-name>/\# -u <username> -P <password> -d
```

