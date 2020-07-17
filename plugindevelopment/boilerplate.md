# Erlang Boilerplate

We recommend to use the `rebar3` toolchain to generate the basic Erlang OTP application boilerplate and start from there.

```text
rebar3 new app name="myplugin" desc="this is my first VerneMQ plugin"
===> Writing myplugin/src/myplugin_app.erl
===> Writing myplugin/src/myplugin_sup.erl
===> Writing myplugin/src/myplugin.app.src
===> Writing myplugin/rebar.config
===> Writing myplugin/.gitignore
===> Writing myplugin/LICENSE
===> Writing myplugin/README.md
```

Change the `rebar.config` file to include the `vernemq_dev` dependency:

```erlang
{erl_opts, [debug_info]}.
{deps, [{vernemq_dev,
    {git, "git://github.com/vernemq/vernemq_dev.git", {branch, "master"}}}
]}.
```

Compile the application, this will automatically fetch `vernemq_dev`.

```text
rebar3 compile                             
===> Verifying dependencies...
===> Fetching vmq_commons ({git,
                                      "git://github.com/vernemq/vernemq_dev.git",
                                      {branch,"master"}})
===> Compiling vernemq_dev
===> Compiling myplugin
```

Now you're ready to implement the hooks. Don't forget to add the proper `vmq_plugin_hooks` entries to your `src/myplugin.app.src` file.

For a complete example, see the [vernemq\_demo\_plugin](https://github.com/vernemq/vernemq_demo_plugin).

