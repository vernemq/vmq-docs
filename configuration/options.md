---
description: Configure how VerneMQ handles certain aspects of MQTT
---

# MQTT Options

## Retry Interval

Set the time in seconds after a `QoS=1 or QoS=2` message has been sent that VerneMQ will wait before retrying when no response is received.

```text
retry_interval = 20
```

This option default to `20` seconds.

## Inflight Messages

This option defines the maximum number of QoS 1 or 2 messages that can be in the process of being transmitted simultaneously.

```text
max_inflight_messages = 20
```

Defaults to `20` messages, use `0` for no limit. The inflight window serves as a protection for sessions, on the incoming side.

## Load Shedding

The maximum number of messages to hold in the queue above those messages that are currently in flight. Defaults to `1000`. Set to `-1` for no limit. This option protects a client session from overload by dropping messages \(of any QoS\).

```text
max_online_messages = 1000
```

Defaults to `1000` messages, use `-1` for no limit. This parameter was named `max_queued_messages` in `0.10.*`. Note that `0` will totally block message delivery from any queue!

This option specifies the maximum number of QoS 1 and 2 messages to hold in the offline queue.

```text
max_offline_messages = 1000
```

Defaults to `1000` messages, use `-1` for no limit, use `0` if no messages should be stored.

In contrast to the session based inflight window, max\_online\_messages and max\_offline\_messages serves as a protection of queues, on the outgoing side.

```text
override_max_online_messages = off
```

When an offline session transists to online, by default VerneMQ will adhere to the queue sizes also for moving data from the offline queue to the online queue. Therefore, if max_offline_messages > max_online_message VerneMQ will start dropping messages. It is possible to override this behaviour and allow VerneMQ to move all messages from the offline queue to the online queue. The queue will then batched (or streamed) to the subscribers, and the messages are read from disk in batches as well. The additional memory needed thus is just the amount needed to store references to those messages and not the messages themselves.
