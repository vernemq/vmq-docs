---
description: Configure VerneMQ Logging.
---

# Logging

## Console Logging

Where should VerneMQ emit the default console log messages \(which are typically at `info` severity\):

```text
log.console = off | file | console | both
```

VerneMQ defaults to log the console messages to a file, which can specified by:

```text
log.console.file = /path/to/log/file
```

This option defaults to `/var/log/vernemq/console.log` for Ubuntu, Debian, RHEL and Docker installs.

The default console logging level `info` could be setting one of the following:

```text
log.console.level = debug | info | warning | error
```

## Error Logging

VerneMQ log error messages by default. One can change the default behaviour by setting:

```text
log.error = on | off
```

VerneMQ defaults to log the error messages to a file, which can specified by:

```text
log.error.file = /path/to/log/file
```

This option defaults to `/var/log/vernemq/error.log` for Ubuntu, Debian, RHEL and Docker installs.

## Crash Logging

VerneMQ log crash messages by default. One can change the default behaviour by setting:

```text
log.crash = on | off
```

VerneMQ defaults to log the crash messages to a file, which can specified by:

```text
log.crash.file = /path/to/log/file
```

This option defaults to `/var/log/vernemq/crash.log` for Ubuntu, Debian, RHEL and Docker installs.

The maximum sizes in bytes of individual messages in the crash log defaults to `64KB` but can be specified by:

```text
log.crash.maximum_message_size = 64KB
```

VerneMQ rotate crash logs. By default, the crash log file is rotated at midnight or when the size exceeds `10MB`. This behaviour can be changed by setting:

```text
## Acceptable values:
##   - a byte size with units, e.g. 10GB
log.crash.size = 10MB

## For acceptable values see https://github.com/basho/lager/blob/master/README.md#internal-log-rotation
log.crash.rotation = $D0
```

The default number of rotated log files is 5 and can be set with the option:

```text
log.crash.rotation.keep = 5
```

## SysLog

VerneMQ supports logging to SysLog, enable it by setting:

```text
log.syslog = on
```

Logging to SysLog is disabled by default.

The minimal SysLog logging level could be set to one of the following (defaults to `all`):

```text
log.syslog.level = all | debug | info | warning | error
```

The SysLog message format defaults to `rfc3164`. Sending messages via TLS requires `rfc5424`:

```text
log.syslog.format = rfc3164 | rfc5424
```

The SysLog facility used for local and remote messages defaults to `daemon`:

```text
log.syslog.facility = daemon
```

Other acceptable facilities are `kern`, `kernel`, `user`, `mail`, `auth`, `syslog`, `lpr`, `news`, `uucp`, `cron`, `authpriv`, `ftp`, `ntp`, `logaudit`, `logalert`, `clock` and `local0` to `local7`.

The application name included in SysLog messages can be set with:

```text
log.syslog.app_name = vernemq
```

If unset, the node name is used.

### Remote SysLog

Setting the remote SysLog server hostname or IP address enables delivery of SysLog messages to a remote host:

```text
log.syslog.remote.host = syslog.example.com
```

The remote SysLog server port defaults to `514`:

```text
log.syslog.remote.port = 514
```

The transport used for remote SysLog delivery defaults to `udp`:

```text
log.syslog.remote.protocol = udp | tcp | tls
```

When `log.syslog.remote.protocol` is set to `tls`, `log.syslog.format` must be `rfc5424` and a CA certificate file must be provided to verify the remote server:

```text
log.syslog.remote.tls.cafile = /path/to/cafile
```

