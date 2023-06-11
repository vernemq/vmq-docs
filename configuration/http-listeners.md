---
description: How to setup and configure the HTTP listener.
---

# HTTP Listeners

The VerneMQ HTTP listener is used to serve various VerneMQ subsystems such as:

* [Status page](../monitoring/status.md)
* [Prometheus metrics](../monitoring/prometheus.md)
* [management API](../live-administration/http-administration.md)
* [Health check](../monitoring/health-check.md)
* [HTTP Publish](http-pub.md)

By default listener runs on port `8888`. To disable the HTTP listener, use a HTTPS listener instead or change the port, adapt the configuration in `vernemq.conf`:

```text
listener.http.default = 127.0.0.1:8888
```

You can have multiple HTTP(s) listener listening to different port and running different modules:
```text
listener.https.default = 127.0.0.1:443
listeners.https.default.http_modules = vmq_status_http, vmq_health_http, vmq_metrics_http

listener.https.mgmt = 127.0.0.1:444
listeners.https.mgmt.http_modules = vmq_mgmt_http
```

This configuration snippet defines two HTTPS listeners with different modules. One for default traffic and one for management traffic. It specifies which HTTP modules will be enabled on each listener, allowing for status, health, and metrics information to be retrieved from the default listener and providing a web-based interface for managing and monitoring VerneMQ through the management listener.
