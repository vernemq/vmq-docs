# Auth using files

## Authentication

VerneMQ comes with a simple file-based password authentication mechanism which is enabled by default. If you don't need this it can be disabled by setting:

```text
plugins.vmq_passwd = off
```

Per default VerneMQ doesn't accept any client that hasn't been configured using `vmq-passwd`. If you want to change this and accept any client connection you can set:

```text
allow_anonymous = on
```

{% hint style="info" %}
Warning: Setting `allow_anonymous=on` completely disables authentication in the broker and plugin authentication hooks are never called! Find more information on the authentication hooks [here](../plugin-development/sessionlifecycle.md#auth_on_register-and-auth_on_register_m5).
{% endhint %}

In a production setup you can use the provided password based authentication mechanism, one of the provided authentication Database plugins, or implement your own authentication plugins.

VerneMQ periodically checks the specified password file.

```text
vmq_passwd.password_file = /etc/vernemq/vmq.passwd
```

The check interval defaults to 10 seconds and can also be defined in the `vernemq.conf`.

```text
vmq_passwd.password_reload_interval = 10
```

Setting the `password_reload_interval = 0` disables automatic reloading.

{% hint style="info" %}
Both configuration parameters can also be changed at runtime using the `vmq-admin` script.

Example: to dynamically set the reload interval to 60 seconds on all your cluster nodes, issue the following command on one of the nodes:

`sudo vmq-admin set vmq_passwd.password_reload_interval=60 --all`
{% endhint %}

### Manage Password Files for VerneMQ

`vmq-passwd` is a tool for managing password files for the VerneMQ broker. Usernames must not contain `":"`, passwords are stored in similar format to [crypt\(3\)](http://man7.org/linux/man-pages/man3/crypt.3.html).

**How to use vmq-passwd**

```text
vmq-passwd [-c | -D] passwordfile username

vmq-passwd -U passwordfile
```

**Options**

`-c`

> Creates a new password file. Does not overwrite existing file.

`-cf`

> Creates a new password file. If the file already exists, it will be overwritten.

`-D`

> Deletes the specified user from the password file.

`-U`

> This option can be used to upgrade/convert a password file with plain text passwords into one using hashed passwords. It will modify the specified file. It does not detect whether passwords are already hashed, so using it on a password file that already contains hashed passwords will generate new hashes based on the old hashes and render the password file unusable. Note, with this option neither usernames or passwords may contain `":"`.

`passwordfile`

> The password file to modify.

`username`

> The username to add/update/delete.

**Examples**

Add a user to a new password file: \(you can choose an arbitrary name for the password file, it only has to match the configuration in the VerneMQ configuration file\).

```text
vmq-passwd -c /etc/vernemq/vmq.passwd henry
```

Delete a user from a password file

```text
vmq-passwd -D /etc/vernemq/vmq.passwd henry
```

**Acknowledgements**

The original version of `vmq-passwd` was developed by Roger Light \(roger@atchoo.org\).

`vmq-passwd` includes :

* software developed by the \[OpenSSL

  Project\]\([http://www.openssl.org/](http://www.openssl.org/)\) for use in the OpenSSL Toolkit.

* cryptographic software written by Eric Young

  \(eay@cryptsoft.com\)

* software written by Tim Hudson \(tjh@cryptsoft.com\)

## Authorization

VerneMQ comes with a simple ACL based authorization mechanism which is enabled by default. If you don't need this it can be disabled by setting:

```text
plugins.vmq_acl = off
```

VerneMQ periodically checks the specified ACL file.

```text
vmq_acl.acl_file = /etc/vernemq/vmq.acl
```

The check interval defaults to 10 seconds and can also be defined in the `vernemq.conf`.

```text
vmq_acl.acl_reload_interval = 10
```

Setting the `acl_reload_interval = 0` disables automatic reloading.

{% hint style="info" %}
Both configuration parameters can also be changed at runtime using the `vmq-admin` script.
{% endhint %}

### Managing the ACL entries

Topic access is added with lines of the format:

```text
topic [read|write] <topic>
```

The access type is controlled using `read` or `write`. If not provided then read an write access is granted for the `topic`. The `topic` can use the MQTT subscription wildcards `+` or `#`.

The first set of topics are applied to all anonymous clients \(assuming `allow_anonymous = on`\). User specific ACLs are added after a user line as follows \(this is the username not the client id\):

```text
user <username>
```

It is also possible to define ACLs based on pattern substitution within the the topic. The form is the same as for the topic keyword, but using pattern as the keyword.

```text
pattern [read|write] <topic>
```

The patterns available for substitution are:

> * `%c` to match the client id of the client
> * `%u` to match the username of the client

The substitution pattern must be the only text for that level of hierarchy. Pattern ACLs apply to all users even if the **user** keyword has previously been given.

Example:

```text
pattern write sensor/%u/data
```

{% hint style="warning" %}
VerneMQ currently doesn't cancel active subscriptions in case the ACL file revokes access for a topic. It is possible to reauthenticate sessions manually (`vmq-admin`)
{% endhint %}

### Simple ACL Example

```text
# ACL for anonymous clients
topic bar
topic write foo
topic read open_to_all


# ACL for user 'john'
user john
topic foo
topic read baz
topic write open_to_all
```

Anonymous users are allowed to

* publish & subscribe to topic bar.
* publish to topic foo.
* subscribe to topic open_to_all.

User john is allowed to

* publish & subscribe to topic foo.
* subscribe to topic baz.
* publish to topic open_to_all.
