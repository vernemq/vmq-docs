---
description: Inspecting the retained message store
---

# Inspecting retained messages
To list the retained messages simply invoke `vmq-admin retain show`:

```shell
$ vmq-admin retain show
+------------------+----------------+
|     payload      |     topic      |
+------------------+----------------+
| a-third-message  | a/third/topic  |
|some-other-message|some/other/topic|
|    a-message     |   some/topic   |
|    a-message     | another/topic  |
+------------------+----------------+
```

{% hint style="success" %}
Note, by default a maximum of 100 results are returned. This is a mechanism to
protect the from overload as there can be millions of retained messages. Use
`--limit=<RowLimit>` to override the default value.
{% endhint %}

Besides listing the retained messages it is also possible to filter them:

```shell
$ vmq-admin retain show --payload --topic=some/topic
+---------+
| payload |
+---------+
|a-message|
+---------+
```

In the above example we list only the payload for the topic `some/topic`.

Another example where all topics are list with retained messages with a specific
payload:

```shell
$ vmq-admin retain show --payload a-message --topic
+-------------+
|    topic    |
+-------------+
| some/topic  |
|another/topic|
+-------------+
```

See the full set of options and documentation by invoking `vmq-admin retain show
--help`.

