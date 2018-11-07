# Non-standard MQTT options

### Maximum Client Id Size

Set the maximum size for client ids, MQTT v3.1 specifies a limit of 23 characters.

```text
max_client_id_size = 23
```

This option default to `23`.

### Persistent Client Expiration

This option allows persistent clients \(those with `clean_session` set to `false`\) to be removed if they do not reconnect within a certain time frame.

{% hint style="warning" %}
This is a non-standard option. As far as the MQTT specification is concerned, persistent clients are persisted forever.
{% endhint %}

The expiration period should be an integer followed by one of `h`, `d`, `w`, `m`, `y` for hour, day, week, month, and year; or `never`:

```text
persistent_client_expiration = 1w
```

This option defaults to `never`.

### Message Size Limit

Limit the maximum publish payload size in bytes that VerneMQ allows. Messages that exceed this size won't be accepted.

```text
message_size_limit = 0
```

Defaults to `0`, which means that all valid messages are accepted. MQTT specification imposes a maximum payload size of 268435455 bytes.

