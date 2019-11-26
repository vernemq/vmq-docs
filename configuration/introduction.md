---
description: Everything you must know to properly configure VerneMQ
---

# Introduction

Every VerneMQ node has to be configured. Depending on the installation method and chosen platform the configuration file `vernemq.conf` resides at different locations. If VerneMQ was installed through a Linux package the default location for the configuration file is `/etc/vernemq/vernemq.conf`.

## File Format

* A single setting is handled on one line.
* Lines are structured `Key = Value`
* Any line starting with \# is a comment, and will be ignored

## Minimal Quickstart configuration

You certainly want to try out VerneMQ right away. For that you could disable authentication like so:

* Set `allow_anonymous = on`

By default the `vmq_acl` authorization plugin is enabled and configured to allow publishing and subscribing to any topic, see [here](file-auth.md##authorization) for more information.

{% hint style="info" %}
Warning: Setting `allow_anonymous=on` completely disables authentication in the broker and plugin authentication hooks are never called! See more information about the authentication hooks [here](../plugindevelopment/sessionlifecycle.md#auth_on_register-and-auth_on_register_m5). Further, in a production system you should configure `vmq_acl` to be less permissive or configure some other plugin to handle authorization.
{% endhint %}

