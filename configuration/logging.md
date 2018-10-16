---
description: Configure VerneMQ Logging.
---

# Logging

### Console Logging

Where should VerneMQ emit the default console log messages \(which are typically at `info` severity\):

```text
log.console = off | file | console | both
```

VerneMQ defaults to log the console messages to a file, which can specified by:

```text
log.console.file = /path/to/log/file
```

This option defaults to the filename `console.log`, whereas the path differs on the way VerneMQ is installed.

The default console logging level `info` could be setting one of the following:

```text
log.console.level = debug | info | warning | error
```

### Error Logging

VerneMQ defaults to log the error messages to a file, which can specified by:

```text
log.error.file = /path/to/log/file
```

This option defaults to the filename `error.log`, whereas the path differs on the way VerneMQ is installed.

### SysLog

VerneMQ supports logging to SysLog, enable it by setting:

```text
log.syslog = on
```

