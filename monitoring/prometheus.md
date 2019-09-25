---
description: Description and Configuration of the Prometheus exporter
---

# Prometheus

The Prometheus exporter is enabled by default and installs an HTTP handler on `http://localhost:8888/metrics`. To read more about configuring the HTTP listener, see [HTTP Listener Configuration](../configuring-vernemq/http-listeners.md).

## Example Scrape Config

Add the following configuration to the `scrape_configs` section inside `prometheus.yml` of your Prometheus server.

```yaml
# A scrape configuration containing exactly one endpoint to scrape: 
# Here it's Prometheus itself.
scrape_configs:
  - job_name: 'vernemq'
    scrape_interval: 5s
    scrape_timeout: 5s
    static_configs:
      - targets: ['localhost:8888']
```

This tells Prometheus to scrape the VerneMQ metrics endpoint every 5 seconds.

Please follow the documentation on the [Prometheus](http://prometheus.io) website to properly configure the metrics scraping as well as how to access those metrics and configure alarms and graphs.

