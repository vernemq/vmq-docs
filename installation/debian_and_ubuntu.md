---
description: >-
  VerneMQ can be installed on Debian or Ubuntu-based systems using the binary
  package we provide.
---

# Installing on Debian and Ubuntu

## Install VerneMQ

Once you have downloaded the binary package, execute the following command to install VerneMQ:

```text
sudo dpkg -i vernemq-<VERSION>.bionic.x86_64.deb
```

## Verify your installation

You can verify that VerneMQ is successfully installed by running:

```text
dpkg -s vernemq | grep Status
```

If VerneMQ has been installed successfully `Status: install ok installed` is returned.

## Activate VerneMQ node

Once you've installed VerneMQ, start it on your node:

```text
service vernemq start
```

## Default Directories and Paths

The `whereis vernemq` command will give you a couple of directories:

```text
whereis vernemq
vernemq: /usr/sbin/vernemq /usr/lib/vernemq /etc/vernemq /usr/share/vernemq
```

| Path | Description |
| :--- | :--- |
| /usr/sbin/vernemq: | the vernemq and vmq-admin commands |
| /usr/lib/vernemq | the vernemq package |
| /etc/vernemq | the vernemq.conf file |
| /usr/share/vernemq | the internal vernemq schema files |
| /var/lib/vernemq | the vernemq data dirs for LevelDB \(Metadata Store and Message Store\) |

## Next Steps

Now that you've installed VerneMQ, check out [How to configure VerneMQ](../configuration/introduction.md).

