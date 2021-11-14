---
description: Managing VerneMQ Plugins
---

# Plugins

Many aspects of VerneMQ can be extended using plugins. The standard VerneMQ package comes with several official plugins. You can show the enabled & running plugins via:

```text
vmq-admin plugin show
```

The command above displays all the enabled plugins together with the hooks they implement:

```text
+-----------+-----------+-----------------+-----------------------------+
|  Plugin   |   Type    |     Hook(s)     |            M:F/A            |
+-----------+-----------+-----------------+-----------------------------+
|vmq_passwd |application|auth_on_register |vmq_passwd:auth_on_register/5|
|  vmq_acl  |application| auth_on_publish |  vmq_acl:auth_on_publish/6  |
|           |           |auth_on_subscribe| vmq_acl:auth_on_subscribe/3 |
+-----------+-----------+-----------------+-----------------------------+
```
The table will show the following information:

- name of the plugin
- type (application or single module)
- all the hooks implemented in the plugin
- the exact module and function names (`M:F/A`) implementing those hooks. 
 
As an example on how to read the table: the `vmq_passwd:auth_on_register/5` function is the actual implementation of the `auth_on_register` hook in the `vmq_passwd` application plugin.

In addition, you can conclude that the plugin is currently running, as it shows up in the table.

To display information on internal plugins, add the `--internal` flag. The table below shows you that the generic metadata application and the generic message store are actually internal plugins. 

```text
$ sudo vmq-admin plugin show --internal
+-----------------------+-------------+-------------------------------+------------------------------------------------+
| Plugin                | Type        | Hook(s)                       | M:F/A                                          |
+-----------------------+-------------+-------------------------------+------------------------------------------------+
| vmq_swc               | application | metadata_put                  | vmq_swc_plugin:metadata_put/3                  |
|                       |             | metadata_get                  | vmq_swc_plugin:metadata_get/2                  |
|                       |             | metadata_delete               | vmq_swc_plugin:metadata_delete/2               |
|                       |             | metadata_fold                 | vmq_swc_plugin:metadata_fold/3                 |
|                       |             | metadata_subscribe            | vmq_swc_plugin:metadata_subscribe/1            |
|                       |             | cluster_join                  | vmq_swc_plugin:cluster_join/1                  |
|                       |             | cluster_leave                 | vmq_swc_plugin:cluster_leave/1                 |
|                       |             | cluster_members               | vmq_swc_plugin:cluster_members/0               |
|                       |             | cluster_rename_member         | vmq_swc_plugin:cluster_rename_member/2         |
|                       |             | cluster_events_add_handler    | vmq_swc_plugin:cluster_events_add_handler/2    |
|                       |             | cluster_events_delete_handler | vmq_swc_plugin:cluster_events_delete_handler/2 |
|                       |             | cluster_events_call_handler   | vmq_swc_plugin:cluster_events_call_handler/3   |
|                       |             |                               |                                                |
+-----------------------+-------------+-------------------------------+------------------------------------------------+
| vmq_generic_msg_store | application | msg_store_write               | vmq_generic_msg_store:msg_store_write/2        |
|                       |             | msg_store_delete              | vmq_generic_msg_store:msg_store_delete/2       |
|                       |             | msg_store_find                | vmq_generic_msg_store:msg_store_find/2         |
|                       |             | msg_store_read                | vmq_generic_msg_store:msg_store_read/2         |
|                       |             |                               |                                                |
+-----------------------+-------------+-------------------------------+------------------------------------------------+
| vmq_config            | module      | change_config                 | vmq_config:change_config/1                     |
|                       |             |                               |                                                |
+-----------------------+-------------+-------------------------------+------------------------------------------------+
| vmq_acl               | application | change_config                 | vmq_acl:change_config/1                        |
|                       |             | auth_on_publish               | vmq_acl:auth_on_publish/6                      |
|                       |             | auth_on_subscribe             | vmq_acl:auth_on_subscribe/3                    |
|                       |             | auth_on_publish_m5            | vmq_acl:auth_on_publish_m5/7                   |
|                       |             | auth_on_subscribe_m5          | vmq_acl:auth_on_subscribe_m5/4                 |
|                       |             |                               |                                                |
+-----------------------+-------------+-------------------------------+------------------------------------------------+
| vmq_passwd            | application | change_config                 | vmq_passwd:change_config/1                     |
|                       |             | auth_on_register              | vmq_passwd:auth_on_register/5                  |
|                       |             | auth_on_register_m5           | vmq_passwd:auth_on_register_m5/6               |
|                       |             |                               |                                                |
+-----------------------+-------------+-------------------------------+------------------------------------------------+
```

## Enable a plugin

```text
vmq-admin plugin enable --name=vmq_acl
```

This enables the ACL plugin. Because the `vmq_acl` plugin is already started the above command won't succeed. In case the plugin sits in an external directory you must also to provide the `--path=PathToPlugin`.

## Disable a plugin

```text
vmq-admin plugin disable --name=vmq_acl
```

## Persisting Plugin Configurations and Starts

To make a plugin start when VerneMQ boots, you need to tell VerneMQ in the main `vernemq.conf` file.

The general syntax to enable a plugin is to add a line like `plugins.pluginname = on`. Using the `vmq_passwd` plugin as an example:

```text
plugins.vmq_passwd = on
```

If the plugin is external (all your own VerneMQ plugin will be of this category), the path can be specified like this:

```text
plugins.myplugin = on
plugins.myplugin.path = /path/to/plugin
```

Plugin specific settings can be configured via `myplugin.somesetting = value`, like:

```text
vmq_passwd.password_file = ./etc/vmq.passwd
```

Check the `vernemq.conf` file for additional details and examples.

