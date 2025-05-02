---
description: The VerneMQ health checker
---

# Health Checker

A simple way to gauge the health of a VerneMQ cluster is to query the `/health` path on the [HTTP listener](../configuration/http-listeners.md).

The health check will return **200** when VerneMQ is accepting connections and is joined with the cluster \(for clustered setups\). **503** will be returned in case any of those two conditions are not met.
In addition to the simple `/health` path, the following options are available as well

- `/health/ping`: Cowboy (ie. Verne) is up.
- `/health/listeners`: will fail if any of the configured listeners is down or suspended
- `/health/listeners_full_cluster`: will fail if any listener is down or any of the cluster nodes is offline. (you probably don't want to use this to base automated actions on the status)

With the `ping` or `listeners` option, you can configure a health check for a single node, even if it is part of a cluster.

If you want to configure any automated actions based on the health check results, you need to chose an appropriate health check path. For example, you should not use the `/health` check (checking for full cluster consistency) to automatically restart a single node. This is of special importance for Kubernetes deployments.
