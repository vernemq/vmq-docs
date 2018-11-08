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

### Enable a plugin

```text
vmq-admin plugin enable --name=vmq_acl
```

This enables the ACL plugin. Because the `vmq_acl` plugin is already started the above command won't succeed. In case the plugin sits in an external directory you must also to provide the `--path=PathToPlugin`.

### Disable a plugin

```text
vmq-admin plugin disable --name=vmq_acl
```

### Persisting plugins

To make a plugin start when VerneMQ starts they need to be configured in the main `vernemq.conf` file.

The general syntax to enable a plugin is to add a line like `plugins.pluginname = on`, using the `vmq_passwd` plugin as an example:

```text
plugins.vmq_passwd = on
```

And if the plugin is external the path can be specified like this:

```text
plugins.myplugin = on
plugins.myplugin.path = /path/to/plugin
```

Plugin specific settings can be configured via `myplugin.somesetting = value`, like:

```text
vmq_passwd.password_file = ./etc/vmq.passwd
```

See the `vernemq.conf` file for details.

