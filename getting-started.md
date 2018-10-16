---
description: A quick and simple guide to get started with VerneMQ
---

# Getting Started

## Installing VerneMQ

VerneMQ is a high-performance, distributed MQTT message broker. It scales horizontally and vertically on commodity hardware to support a high number of concurrent publishers and consumers while maintaining low latency and fault tolerance. To use it, all you need to do is install the VerneMQ package.

Choose your OS and follow the instructions:

* [CentOS/RHEL 7](centos_and_redhat.html)
* [Debian Stretch & Jessie](debian_and_ubuntu.html)
* [Ubuntu Bionic, Xenial & Trusty](debian_and_ubuntu.html)

It is also possible to run VerneMQ using our Docker image:

* [Docker](docker.html)

## Starting VerneMQ

{% hint style="info" %}
If you built VerneMQ from sources, you can add the `/bin` directory of your VerneMQ release to `PATH`. For example, if you compiled VerneMQ in the `/home/vernemq` directory, then add the binary directory \(`/home/vernemq/_build/default/rel/vernemq/bin`\) to your PATH, so that VerneMQ commands can be used in the same manner as with a packaged installation.
{% endhint %}

To start a VerneMQ broker, use the vernemq start command in your Shell:

```text
vernemq start
```

A successful start will return no output. If there is a problem starting the broker, an error message is printed to `STDERR`.

To run VerneMQ with an attached interactive Erlang console:

```text
vernemq console
```

A VerneMQ broker is typically started in console mode for debugging or troubleshooting purposes. Note that if you start VerneMQ in this manner, it is running as a foreground process that will exit when the console is closed.

You can close the console by issuing this command at the Erlang prompt:

```text
q().
```

Once your broker has started, you can initially check that it is running with the vernemq ping command:

```text
vernemq ping
```

The command will respond with `pong` if the broker is running or `Node <NodeName> not responding to pings` in case it’s not.

{% hint style="warning" %}
As you may have noticed, VerneMQ will warn you at startup when your system’s open files limit \(`ulimit -n`\) is too low. You’re advised to increase the OS default open files limit when running VerneMQ. Read more about why and how in the [Open Files Limit documentation](https://github.com/vernemq/vmq-docs/tree/8e76a28412d2512e71653e1356f7165aebe687f7/docs/misc/change-open-file-limits.html).
{% endhint %}



