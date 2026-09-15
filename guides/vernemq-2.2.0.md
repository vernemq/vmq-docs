# VerneMQ 2.2.0

VerneMQ 2.2.0 adds runtime compatibility updates, listener-level configuration improvements, performance options, and operational examples. This page summarizes the most relevant user-facing changes and links to the detailed documentation.

## Runtime Support

VerneMQ 2.2.0 supports running on Erlang/OTP 28 and Erlang/OTP 29.

## Fanout Sharding

Fanout sharding can increase throughput and lower backpressure for high-frequency publishers where local topic fanout has to deliver to many subscribers. The related advanced settings are `fanout.shard_count` and `fanout.async_handoff`.

See [Advanced Options](../configuration/advanced_options.md#fanout-sharding).

## Subscription Limit Guardrail

The new `max_subscriptions_per_client` setting limits how many subscriptions a client can have. A value of `0` keeps the default unlimited behavior.

See [MQTT Options](../configuration/options.md#subscriptions-per-client).

## Per-Listener Auth Chains

MQTT listeners can define their own authentication and authorization plugin chains with `auth_plugins` and `authz_plugins`. This allows different listeners to use different authentication and authorization behavior without changing the global plugin chain.

See [MQTT Listeners](../configuration/listeners.md#per-listener-authentication-and-authorization) and [Managing Listeners](../administration/listeners.md#authentication-and-authorization-plugin-chains).

## Forward Connection Options

`forward_connection_opts` can pass listener metadata to `auth_on_register` and `auth_on_register_m5` hooks. Plugins can use that metadata to implement different behavior based on the listener that accepted the client connection.

See [MQTT Listeners](../configuration/listeners.md#forward-connection-options) and [Session lifecycle](../plugindevelopment/sessionlifecycle.md#listener-metadata).

## Listener active_n

TCP and SSL listeners support `active_n`, which controls how many incoming TCP packets may be delivered before the socket switches back to passive mode. The default is `1`, preserving the previous behavior.

See [MQTT Listeners](../configuration/listeners.md#active-socket-packets).

## WebSocket Anonymous Override

`allow_anonymous_override` is now available for WebSocket and secure WebSocket listeners as well as TCP and SSL listeners. This allows a named listener to override a global `allow_anonymous = off` setting.

See [MQTT Listeners](../configuration/listeners.md#anonymous-access-override).

## Cluster and Internode Robustness

VerneMQ 2.2.0 improves internode MQTT delivery with a connect-ack handshake, bounded inbound buffering, outgoing batching, and larger default outgoing clustering buffers. These settings are advanced and should usually be left at their defaults.

See [Inter-node Communication](../clustering/communication.md#internode-mqtt-delivery).

## Prometheus Alert Rules Template

VerneMQ ships with a minimal Prometheus alert rules template in `metrics_scripts/prometheus/vernemq-alert-rules.yml`. The template is intended as a starting point and should be extended or adapted to the thresholds, labels, severities, and runbooks of each deployment.

See [Prometheus](../monitoring/prometheus.md#example-alert-rules).
