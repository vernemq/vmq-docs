

description: Changing the output format of CLI commands


## Default Output Format

The default output format is called `human-readable`. It will print tables or text answers in response to your CLI commands.


## JSON Output Format

The only alternative format is JSON. You can request it by adding the `--format=json` key to a command.


```
vmq-admin listener show --format=json
```

```
{"table":[{"type":"vmq","status":"running","address":"0.0.0.0","port":"44053","mountpoint":"","max_conns":10000,"active_conns":0,"all_conns":0},{"type":"mqtt","status":"running","address":"127.0.0.1","port":"1883","mountpoint":"","max_conns":10000,"active_conns":0,"all_conns":0},{"type":"mqttws","status":"running","address":"127.0.0.1","port":"1887","mountpoint":"","max_conns":10000,"active_conns":0,"all_conns":0},{"type":"http","status":"running","address":"127.0.0.1","port":"8888","mountpoint":"","max_conns":10000,"active_conns":0,"all_conns":0}],"type":"table"}%
```

To pretty-print your JSON or extract the `table` object, use the `jq` command. Currently, not all responses give you a nice table and attributes format. Namely, `vmq-admin metrics show` will only give the metrics as text.

```
vmq-admin listener show --format=json | jq '.table'
```

```json
[
  {
    "type": "vmq",
    "status": "running",
    "address": "0.0.0.0",
    "port": "44053",
    "mountpoint": "",
    "max_conns": 10000,
    "active_conns": 0,
    "all_conns": 0
  },
  {
    "type": "mqtt",
    "status": "running",
    "address": "127.0.0.1",
    "port": "1883",
    "mountpoint": "",
    "max_conns": 10000,
    "active_conns": 0,
    "all_conns": 0
  },
  {
    "type": "mqttws",
    "status": "running",
    "address": "127.0.0.1",
    "port": "1887",
    "mountpoint": "",
    "max_conns": 10000,
    "active_conns": 0,
    "all_conns": 0
  },
  {
    "type": "http",
    "status": "running",
    "address": "127.0.0.1",
    "port": "8888",
    "mountpoint": "",
    "max_conns": 10000,
    "active_conns": 0,
    "all_conns": 0
  }
]
```