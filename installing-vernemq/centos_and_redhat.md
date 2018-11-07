# Installing on CentOS and RHEL

### Install VerneMQ

Once you have downloaded the binary package, execute the following command to install VerneMQ:

```text
sudo yum install vernemq-<%= latest_version() %>-1.el7.centos.x86_64.rpm
```

or:

```text
sudo rpm -Uvh vernemq-<%= latest_version() %>-1.el7.centos.x86_64.rpm
```

### Activate VerneMQ node

Once you've installed VerneMQ, start it on your node:

```text
service vernemq start
```

### Verify your installation

You can verify that VerneMQ is successfully installed by running:

```text
rpm -qa | grep vernemq
```

If VerneMQ has been installed successfully `vernemq` is returned.

### Next Steps

Now that you've installed VerneMQ, check out [How to configure VerneMQ](../configuring-vernemq/introduction.md).

