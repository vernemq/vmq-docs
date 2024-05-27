---
description: Schema Files in VerneMQ
---

## Multiple Schema files, one VerneMQ conf file

During every boot up, VerneMQ will run your `vernemq.conf` file against the Schema files of the VerneMQ release. This serves as a validation and as a mechanism to create the timestamped internal config files that you'll find in the `generated.configs` directory.

In general, every application of the VerneMQ release has its own schema file in the `priv` subdirectory (the only exception is the `vmq.schema` file in the `file` directory). A `my_app.schema` defines all the configuration settings you can use for that application.

And that's almost the only reason to know a bit about schema files: you can browse them for possible settings if you suspect a minor settings is not yet fully documented. Most of the time you'll also find at least a short snippet documenting the setting in the schema.file.

An example from the `vmq_server.schema`:

```
%% @doc specifies the max duration of an API key before it expires (default: undefined)
{mapping, "max_apikey_expiry_days", "vmq_server.max_apikey_expiry_days", 
                                            [{datatype, integer}, 
                                            {default, undefined},
                                            hidden
                                                                                ]}.
```

This is a relatively minor feature where you can set a default expiry for API keys. You can determine from the mapping schema that a default is not set. To set the value in the `vernemq.conf` file, always use the left-hand name from the mapping in the schema:

```
max_apikey_expiry_days = 30
```

You can also see the keyword `hidden` in the mapping. This means that the setting will not show up automatically in the `vernemq.conf` file and you'll have to add the it manually.