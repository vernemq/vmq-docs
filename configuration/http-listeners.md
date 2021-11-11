---
description: How to setup and configure the HTTP listener.
---

# HTTP Listeners

The VerneMQ HTTP listener is used to serve various VerneMQ subsystems such as:

* [Status page](../monitoring/status.md)
* [Prometheus metrics](../monitoring/prometheus.md)
* [management API](../live-administration/http-administration.md)
* [Health check](../monitoring/health-check.md)

By default it runs on port `8888`. To disable the HTTP listener or change the port, adapt the configuration in `vernemq.conf`:

```text
listener.http.default = 127.0.0.1:8888
```

