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

