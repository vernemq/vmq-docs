---
description: Description and Configuration of the built-in Monitoring mechanism
---

# Introduction

VerneMQ can be monitored in several ways. We implemented native support for [Graphite](https://graphiteapp.org/), [MQTT $SYS tree](systree.md), and [Prometheus](http://prometheus.io).

The metrics are also available via the command line tool:

```text
vmq-admin metrics show
```

### Exported Metrics

VerneMQ metrics are either counters or gauges. Counters always report the total number of events since the broker start. Gauges report a point in time value.

#### Available Counters

```text
# Networking
socket_open 
socket_close 
socket_error 
bytes_received 
bytes_sent 

# Inter-node messages, only includes the distribution of MQTT messages
cluster_bytes_received 
cluster_bytes_sent 
cluster_bytes_dropped 

# MQTT message stats
mqtt_connect_received 
mqtt_publish_received 
mqtt_puback_received 
mqtt_pubrec_received
mqtt_pubrel_received 
mqtt_pubcomp_received 
mqtt_subscribe_received 
mqtt_unsubscribe_received 
mqtt_pingreq_received 
mqtt_disconnect_received 
mqtt_publish_sent 
mqtt_puback_sent 
mqtt_pubrec_sent 
mqtt_pubrel_sent 
mqtt_pubcomp_sent
mqtt_suback_sent 
mqtt_unsuback_sent 
mqtt_pingresp_sent 

# Authentication stats
mqtt_connack_accepted_sent
mqtt_connack_unacceptable_protocol_sent
mqtt_connack_identifier_rejected_sent
mqtt_connack_server_unavailable_sent
mqtt_connack_bad_credentials_sent
mqtt_connack_not_authorized_sent

# Authorization error
mqtt_publish_auth_error 
mqtt_subscribe_auth_error 

# MQTT stack errors
mqtt_publish_invalid_msg_size_error 
mqtt_puback_invalid_error 
mqtt_pubrec_invalid_error 
mqtt_pubcomp_invalid_error 
mqtt_connect_error 
mqtt_publish_error
mqtt_subscribe_error 
mqtt_unsubscribe_error 

# MQTT queue stats
queue_setup 
queue_teardown 
queue_message_drop 
queue_message_unhandled 
queue_message_in 
queue_message_out 

# MQTT Misc
client_expired 

# System Counters
system_wallclock
system_runtime
system_reductions 
system_io_out 
system_io_in 
system_words_reclaimed_by_gc 
system_gc_count 
system_exact_reductions 
system_context_switches
```

#### Available Gauges

```text
# MQTT Queues
queue_processes

# Retain Cache
retain_memory
retain_messages

# Routing Tables
router_memory
router_topics
router_subscriptions

# System Stats
system_utilization_scheduler_[1..n]
system_utilization
system_run_queue

# VM Memory
vm_memory_ets
vm_memory_code
vm_memory_binary
vm_memory_atom_used
vm_memory_atom
vm_memory_system
vm_memory_processes_used
vm_memory_processes
vm_memory_total
```

