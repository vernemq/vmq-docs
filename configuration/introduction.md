---
description: Everything you must know to properly configure VerneMQ
---

# Introduction

Every VerneMQ node has to be configured as the default configuration probably does not match your needs. Depending on the installation method and chosen platform the configuration file `vernemq.conf` resides at different locations. If VerneMQ was installed through a Linux package the default location for the configuration file is `/etc/vernemq/vernemq.conf`.

## General Format of the `vernemq.conf` file

* A single setting is handled on one line.
* Lines are structured `Key = Value`
* Any line starting with \# is a comment, and will be ignored.

## Minimal Quickstart Configuration

You certainly want to try out VerneMQ right away. To just check the broker without configured authentication for now, you can allow anonymous access:

* Set `allow_anonymous = on`

By default the `vmq_acl` authorization plugin is enabled and configured to allow publishing and subscribing to any topic (basically allowing everything), check the [section on file-based authorization](file-auth.md##authorization) for more information.

{% hint style="warning" %}
Setting `allow_anonymous=on` completely disables authentication in the broker and plugin authentication hooks are never called! Find the details on all the authentication hooks [here](../plugin-development/sessionlifecycle.md#auth_on_register-and-auth_on_register_m5). **In a production system you should configure `vmq_acl` to be less permissive or configure some other plugin to handle authorization**.
{% endhint %}

