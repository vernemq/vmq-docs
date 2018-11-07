# HTTP API

{% hint style="warning" %}
This API is still in BETA and some things may change if required.
{% endhint %}

The VerneMQ HTTP API is enabled by default and installs an HTTP handler on `http://localhost:8888/api/v1`. To read more about configuring the HTTP listener, see [HTTP Listener Configuration](../configuring-vernemq/http-listeners.md). You can configure a HTTP listener, or a HTTPS listener to serve the HTTP API v1.

## Managing API keys

The VerneMQ HTTP API uses basic authentication where an API key is passed as the username and the password is left empty. So the first step to us the HTTP API is to create an API key:

```text
$ vmq-admin api-key create
JxctXkZ1OTVnlwvguSCE9KtujacMkOLF
```

The key is persisted and available on all cluster nodes.

To list existing keys do:

```text
$ vmq-admin api-key show
+--------------------------------+
|              Key               |
+--------------------------------+
|JxctXkZ1OTVnlwvguSCE9KtujacMkOLF|
+--------------------------------+
```

To add an API key of your own choosing, do:

```text
vmq-admin api-key add key=mykey
```

To delete an API key do:

```text
vmq-admin api-key delete key=JxctXkZ1OTVnlwvguSCE9KtujacMkOLF
```

## API usage

The VerneMQ HTTP API is a wrapper over the [`vmq-admin`](introduction.md) CLI tool, and anything that can be done using `vmq-admin` can be done using the HTTP API. Note that the HTTP API is therefore subject to any changes made to the `vmq-admin` tools and their flags & options structure. All requests are performed doing a HTTP GET and if no errors occurred an HTTP 200 OK code is returned with a possible non-empty JSON payload.

The API is using basic auth where the API key is passed as the username. An example using `curl` would look like this:

```text
curl "http://JxctXkZ1OTVnlwvguSCE9KtujacMkOLF@localhost:8888/api/v1/session/show"
```

The mapping between `vmq-admin` and the HTTP API is straightforward, and if one is already familiar with how the `vmq-admin` tool works, working with the API should be easy. The mapping works such that the command part of a `vmq-admin` invocation is turned into a path, and the options and flags are turned into the query string.

A mandatory parameter like the `client-id` in the `vmq-admin session disconnect client-id=myclient` command should be translated as: `?client-id=myclient`.

An optional flag like `--cleanup` in the `vmq-admin session disconnect client-id=myclient --cleanup` command should be translated as: `&--cleanup`

Let's look at the cluster join command as an example, which looks like this:

```text
vmq-admin cluster join discovery-node=NodeB@10.0.0.2
```

This turns into a GET request:

```text
GET /api/v1/cluster/join?discovery-node=NodeB@10.0.0.2
```

To test, run it with `curl`:

```text
curl "http://JxctXkZ1OTVnlwvguSCE9KtujacMkOLF@localhost:8888/api/v1/cluster/join?discovery-node=NodeB@10.0.0.2"
```

And the returned response would look like:

```javascript
{
    "text": "Done",
    "type": "text"
}
```

Below are some other examples.

### Get cluster status information

Request:

```text
GET /api/v1/cluster/show
```

Curl:

```text
curl "http://JxctXkZ1OTVnlwvguSCE9KtujacMkOLF@localhost:8888/api/v1/cluster/show"
```

Response:

```javascript
{
   "type" : "table",
   "table" : [
      {
         "Running" : true,
         "Node" : "VerneMQ@127.0.0.1"
      }
   ]
}
```

### Retrieve session information

Request:

```text
GET /api/v1/session/show
```

Curl:

```text
curl "http://JxctXkZ1OTVnlwvguSCE9KtujacMkOLF@localhost:8888/api/v1/session/show"
```

Response:

```javascript
{
   "type" : "table",
   "table" : [
      {
         "user" : null,
         "client_id" : "mosqsub/11690-algol",
         "peer_host" : "localhost",
         "pid" : "<0.551.0>",
         "peer_port" : 50964
      },
      {
         "peer_host" : "localhost",
         "pid" : "<0.557.0>",
         "peer_port" : 50996,
         "user" : null,
         "client_id" : "mosqsub/11767-algol"
      }
   ]
}
```

### List all installed listeners

Request:

```text
GET /api/v1/listener/show
```

Curl:

```text
curl "http://JxctXkZ1OTVnlwvguSCE9KtujacMkOLF@localhost:8888/api/v1/listener/show"
```

Response:

```javascript
{
   "type" : "table",
   "table" : [
      {
         "max_conns" : 10000,
         "port" : "8888",
         "mountpoint" : "",
         "ip" : "127.0.0.1",
         "type" : "http",
         "status" : "running"
      },
      {
         "status" : "running",
         "max_conns" : 10000,
         "port" : "44053",
         "mountpoint" : "",
         "ip" : "0.0.0.0",
         "type" : "vmq"
      },
      {
         "max_conns" : 10000,
         "port" : "1883",
         "mountpoint" : "",
         "ip" : "127.0.0.1",
         "type" : "mqtt",
         "status" : "running"
      }
   ]
}
```

### Retrieve plugin information

Request:

```text
GET /api/v2/plugin/show
```

Curl:

```text
curl "http://JxctXkZ1OTVnlwvguSCE9KtujacMkOLF@localhost:8888/api/v1/plugin/show"
```

Response:

```javascript
    {
   "type" : "table",
   "table" : [
      {
         "Hook(s)" : "auth_on_register\n",
         "Plugin" : "vmq_passwd",
         "M:F/A" : "vmq_passwd:auth_on_register/5\n",
         "Type" : "application"
      },
      {
         "Type" : "application",
         "M:F/A" : "vmq_acl:auth_on_publish/6\nvmq_acl:auth_on_subscribe/3\n",
         "Plugin" : "vmq_acl",
         "Hook(s)" : "auth_on_publish\nauth_on_subscribe\n"
      }
   ]
}
```

### Set configuration values

Request:

```text
GET /api/v1/set?allow_anonymous=on
```

Curl:

```text
curl "http://JxctXkZ1OTVnlwvguSCE9KtujacMkOLF@localhost:8888/api/v1/set?allow_anonymous=on"
```

Response:

```javascript
[]
```

### Disconnect a client

Request:

```text
GET /api/v1/session/disconnect?client-id=myclient&--cleanup
```

Curl:

```text
curl "http://JxctXkZ1OTVnlwvguSCE9KtujacMkOLF@localhost:8888/api/v1/session/disconnect?client-id=myclient&--cleanup"
```

Response:

```javascript
[]
```

