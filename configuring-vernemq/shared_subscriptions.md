# Shared subscriptions

A shared subscription is a mechanism for distributing messages to a set of subscribers to shared subscription topic, such that each message is received by only one subscriber. This contrasts with normal subscriptions where each subscriber will receive a copy of the published message.

A shared subscription is on the form `$share/sharename/topic` and subscribers to this topic will receive messages published to the topic `topic`. The messages will be distributed according to the defined distribution policy.

{% hint style="success" %}
When subscribing to a shared subscription using command line tools remember to quote the topic as some command line shells, like `bash`, will otherwise expand the `$share` part of the topic as an environment variable.
{% endhint %}

## Configuration

Currently three message distribution policies for shared subscriptions are supported: `prefer_local`, `random` and `local_only`. Under the `random` policy messages will be published to a random member of the shared subscription, if any exist. Under the `prefer_local` policy messages will be delivered to a random node-local member of the shared subscription, if none exist, the message will be delivered to a random member of the shared subscription on a remote cluster node. Under the `local_only` policy message will be delivered to a random node-local member of the shared subscription.

```text
shared_subscription_policy = prefer_local
```

When a messages is being delivered to subscribers of a shared subscription, the message will be delivered to an online subscriber if possible, otherwise the message will be delivered to an offline subscriber.

