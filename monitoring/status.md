---
description: The VerneMQ status page
---

# Status Page

VerneMQ comes with a built-in status page that is enabled by default and is available on `http://localhost:8888/status`, see [HTTP listeners](../configuring-vernemq/http-listeners.md).

The status page is a simple overview of the cluster and the individual nodes in the cluster as seen below. Note that while the Status Page is running on each node of the cluster, it's enough to look at one of them to get a quick status of your cluster.

The Status Page has the following sections:

- Issues (Warnings on netsplits, etc)
- Cluster Overview
- Node Status



![](../.gitbook/assets/vernemq_status_page.png)

