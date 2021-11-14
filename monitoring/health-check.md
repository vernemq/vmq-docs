---
description: The VerneMQ health checker
---

# Health Checker

A simple way to gauge the health of a VerneMQ cluster is to query the `/health` path on the [HTTP listener](../configuration/http-listeners.md).

The health check will return **200** when VerneMQ is accepting connections and is joined with the cluster \(for clustered setups\). **503** will be returned in case any of those two conditions are not met.

