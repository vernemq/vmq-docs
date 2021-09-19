---
description: >-
  VerneMQ can be installed on CentOS-based systems using the binary package we
  provide.
---

# Installing on CentOS and RHEL

## Install VerneMQ

Once you have downloaded the binary package, execute the following command to install VerneMQ:

```text
sudo yum install vernemq-<VERSION>.centos7.x86_64.rpm
```

or:

```text
sudo rpm -Uvh vernemq-<VERSION>.centos7.x86_64.rpm
```

## Activate VerneMQ node

{% hint style="danger" %}
To use the provided binary packages the VerneMQ EULA must be accepted. See [Accepting the VerneMQ EULA](accepting-the-vernemq-eula.md) for more information.
{% endhint %}

Once you've installed VerneMQ, start it on your node:

```text
service vernemq start
```

## Verify your installation

You can verify that VerneMQ is successfully installed by running:

```text
rpm -qa | grep vernemq
```

If VerneMQ has been installed successfully `vernemq` is returned.

## Next Steps

Now that you've installed VerneMQ, check out [How to configure VerneMQ](../configuration/introduction.md).

