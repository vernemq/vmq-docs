---
description: Loadtesting VerneMQ with vmq_mzbench
---

# Loadtesting VerneMQ

You can loadtest VerneMQ with our [vmq\_mzbench tool](https://github.com/erlio/vmq_mzbench). It is based on Machinezone's very powerful [MZBench system](https://github.com/satori-com/mzbench) and lets you narrow down what hardware specs are needed to meet your performance goals. You can state your requirements for latency percentiles \(and much more\) in a formal way, and let vmq\_mzbench automatically fail, if it can't meet the requirements.

If you have an AWS account, vmq\_mzbench can automagically provision worker nodes for you. You can also run it locally, of course.

## 1. Install MZBench

Please follow the [MZBench installation guide](http://satori-com.github.io/mzbench/#installation)

## 2. Install vmq\_mzbench

Actually, you don't even have to install vmq\_mzbench, if you don't want to. Your scenario file will automatically fetch vmq\_mzbench for any test you do. vmq\_mzbench runs every test independently, so it has a provisioning step for any test, even if you only run it on a local worker.

To install vmq\_mzbench on your computer, go through the following steps:

```text
git clone git://github.com/erlio/vmq_mzbench.git
cd vmq_mzbench
./rebar get-deps
./rebar compile
```

To provision your tests from this local repository, you'll have to tell the scenario scripts to use rsync. Add this to the scenario file:

```erlang
{make_install, [
{rsync, "/path/to/your/installation/vmq_mzbench/"},
{exclude, "deps"}]},
```

If you'd just like the script itself fetch vmq\_mzbench, then you can direct it to github:

```erlang
{make_install, [
{git, "git://github.com/erlio/vmq_mzbench.git"}]},
```

## 3. Write vmq\_mzbench scenario files

{% hint style="info" %}
MZBench recently switched from an Erlang-styled Scenario DSL to a more python-like DSL dubbed BDL \(Benchmark Definition Language\). Have a look at the [BDL examples](https://github.com/machinezone/mzbench/tree/master/examples.bdl) on Github.
{% endhint %}

You can familiarize yourself quickly with [MZBench's guide](http://satori-com.github.io/mzbench/scenarios/spec/) on writing loadtest scenarios.

There's not much to learn, just make sure you understand how pools and loops work. Then you can add the vmq\_mzbench statement functions to the mix and define actual loadtest scenarios.

Currently vmq\_mzbench exposes the following statement functions for use in MQTT scenario files:

* `random_client_id(State, Meta, I)`: Create a random client Id of length I
* `fixed_client_id(State, Meta, Name, Id)`: Create a deterministic client Id with schema Name ++ "-" ++ Id
* `worker_id(State, Meta)`: Get the internal, sequential worker Id
* `client(State, Meta)`: Get the client Id you set yourself during connection setup with the option {t, client, "client"}
* `connect(State, Meta, ConnectOpts)`: Connect to the broker with the options given in ConnectOpts
* `disconnect(State, Meta)`: Disconnect normally
* `subscribe(State, Meta, Topic, QoS)`: Subscribe to Topic with Quality of Service QoS
* `unsubscribe(State, Meta, Topic)`: Unubscribe from Topic
* `publish(State, Meta, Topic, Payload, QoS)`: Publish a message with binary Payload to Topic with QoS
* `publish(State, Meta, Topic, Payload, QoS, RetainFlag)`: Publish a message with binary Payload to Topic with QoS and RetainFlag

It's easy to add more statement functions to the MQTT worker if needed, get in touch with us.

