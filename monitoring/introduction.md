---
description: Description and Configuration of the built-in Monitoring mechanism
---

# Introduction

VerneMQ can be monitored in several ways. We implemented native support for [Graphite](https://graphiteapp.org/), [MQTT $SYS tree](systree.md), and [Prometheus](http://prometheus.io).

The metrics are also available via the command line tool:

```text
vmq-admin metrics show
```

Or with:

```text
vmq-admin metrics show -d
```

Which will output the metrics together with a short description describing what
the metric is about. An example looks like:

```text
# The number of AUTH packets received.
counter.mqtt_auth_received = 0

# The number of times a MQTT queue process has been initialized from offline storage.
counter.queue_initialized_from_storage = 0

# The number of PUBLISH packets sent.
counter.mqtt_publish_sent = 10

# The number of bytes used for storing retained messages.
gauge.retain_memory = 21184
```

Notice that the metrics:

```text
mqtt_connack_not_authorized_sent
mqtt_connack_bad_credentials_sent
mqtt_connack_server_unavailable_sent
mqtt_connack_identifier_rejected_sent
mqtt_connack_unacceptable_protocol_sent
mqtt_connack_accepted_sent
```

Are no longer used (always 0) and will be removed in the future. They were
replaced with `mqtt_connack_sent` using the `return_code` label. For MQTT 5.0
the `reason_code` label is used instead.
