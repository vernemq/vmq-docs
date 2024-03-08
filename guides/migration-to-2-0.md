# Migrating to 2.0.0 from Previous VerneMQ versions

Release 2.0.0 has a small number of minor incompatibilities:

### Error Logger

VerneMQ now uses the internal logger library instead of the lager library. It's best for your custom VerneMQ plugins to do the same and replace the lager log calls with internal log statements. Instead of using lager:error/2, you can use the following format:

```
?LOG_ERROR("an error happened because: ~p", [Reason]).   % With macro
logger:error("an error happened because: ~p", [Reason]). % Without macro
```

To use the Logger Macros, add this include line to your module: -include_lib("kernel/include/logger.hrl").

### Removed Features

 - The multiple sessions feature has been fully removed. (you are likely not affected by this)
 - Compatibility to and old (v0.15) subscriber format was removed. (you are likely not affected by this)
    
### on_deliver hook

The `on_deliver` hook now has a Properties argument like the `on_deliver_m5` hook. This changes the function arity from `on_deliver/6` to `on_deliver/7`. You can ignore the Properties argument in your on_deliver hook implementation, but you'll have to adapt the function definition, by adding a variable similar to:

```
on_deliver(UserName, SubscriberId, QoS, Topic, Payload, IsRetain, _Properties) ->
 ...
```

### General note
Some settings related to logging were adapted a bit, and there are additional settings exposed in the vernemq.conf file. The Linux package installer gives you the choice to use an existing `vernemq.conf` file, or start with a new template. Depending on the number of settings you have changed, it might be easiest to to move and safe your old `vernemq.conf`, and then use the new template to re-add your settings.