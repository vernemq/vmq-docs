# Graphite

The graphite exporter reports the broker metrics at a fixed interval \(defined in milliseconds\) to a graphite server. The necessary configuration is done inside the `vernemq.conf`.

```text
graphite_enabled = on
graphite_host = carbon.hostedgraphite.com
graphite_port = 2003
graphite_interval = 20000
graphite_api_key = YOUR-GRAPHITE-API-KEY
graphite.interval = 15000
graphite.host = carbon.hostedgraphite.com
graphite.port = 2003
graphite.api_key = YOUR-GRAPHITE-API-KEY
```

You can further tune the connection to the Graphite server:

```text
# set the connect timeout (defaults to 5000 ms)
graphite_connect_timeout = 10000

# set a reconnect timeout (default to 15000 ms)
graphite_reconnect_timeout = 10000

# set a custom graphite prefix (defaults to '')
graphite_prefix = vernemq
```

{% hint style="info" %}
The above configuration parameters can be changed at runtime using the `vmq-admin` script.  
 Usage: `vmq-admin set = ... [[--node | -n]  | --all]`   
 Example: `vmq-admin set graphite_interval=20000 graphite_port=2003 -n VerneMQ@127.0.0.1`
{% endhint %}

