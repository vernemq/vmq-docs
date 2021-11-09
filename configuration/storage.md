# Storage

VerneMQ uses Google's LevelDB as a fast storage backend for messages and subscriber information. Each VerneMQ node runs its own embedded LevelDB store.

### Configuration of LevelDB memory

There's not much you need to know about LevelDB and VerneMQ. One really important thing to note is that LevelDB manages its own memory. This means that VerneMQ will not allocate and free memory for LevelDB. Instead you'll have to configure a configuration value in vernemq.conf that tells LevelDB how much memory it can use up.

Configuring LevelDB memory:

```text
leveldb.maximum_memory.percent = 20
```

{% hint style="danger" %}
LevelDB means business with its allocated memory. It will eventually end up with the configured max, making it look like there's a memory leak, or even triggering OOM kills. Keep that in mind when configuring the percentage of RAM you give to LevelDB.
{% endhint %}

