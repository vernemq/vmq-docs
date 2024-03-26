---
description: A quick and simple guide to get started with VerneMQ
---

# Getting Started

## Installing VerneMQ

VerneMQ is a high-performance, distributed MQTT message broker. It scales horizontally and vertically on commodity hardware to support a high number of concurrent publishers and consumers while maintaining low latency and fault tolerance. To use it, all you need to do is install the VerneMQ package.

Choose your OS and follow the instructions:

* [CentOS/RHEL](installation/centos_and_redhat.md)
* [Debian/Ubuntu](installation/debian_and_ubuntu.md)

It is also possible to run VerneMQ using our Docker image:

* [Docker](installation/docker.md)

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
As you may have noticed, VerneMQ will warn you at startup when your system’s open files limit \(`ulimit -n`\) is too low. You’re advised to increase the OS default open files limit when running VerneMQ. Read more about why and how in the [Open Files Limit documentation](guides/change-open-file-limits.md).
{% endhint %}

### Starting using systemd/systemctl

If you use a `systemd` service file (as in the binary packages), you can start VerneMQ using the `systemctl` interface to `systemd`:

```text
$ sudo systemctl start vernemq
```

Other `systemctl` commands work as well:

```text
$ sudo systemctl stop vernemq
$ sudo systemctl status vernemq
```
