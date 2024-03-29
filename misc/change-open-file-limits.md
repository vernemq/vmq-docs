---
description: How to change the open file limits
---

# Change Open File Limits

VerneMQ can consume a large number of open file handles when thousands of clients are connected as every connection requires at least one file handle.

Most operating systems can change the open-files limit using the `ulimit -n` command. Example:

```text
ulimit -n 262144
```

However, this only changes the limit for the _**current shell session**_. Changing the limit on a system-wide, permanent basis varies more between systems.

What will actually happen when VerneMQ runs out of OS-side file descriptors? 

In short, VerneMQ will be unable to function properly, because it can't open database files or accept incoming connections. 
In case you see exceptions with `{error,emfile}` in the VerneMQ log files, you now know what to do, though: increase the OS settings as described below.

## Linux

On most Linux distributions, the total limit for open files is controlled by `sysctl`.

```text
sysctl fs.file-max
fs.file-max = 262144
```
An alternative way to read the `file-max` settings is:

```text
cat /proc/sys/fs/file-max
```

This might be high enough for your VerneMQ deployment, or not - we cannot know that. You will need at least 1 file descriptor per TCP connection, and VerneMQ needs additional file descriptors for file access etc. Also, if you have other components running on the system, you might want to consult the [sysctl manpage](http://linux.die.net/man/8/sysctl) manpage for how to change that setting. The `fs.file-max` setting represents the global maximum of file handlers a Linux kernel will allocate. Make sure this is high enough for your system.

Once you're good regarding `file-max`, you still need to configure the per-process open files limit. You'll set the number of file descriptors a single process or application like VerneMQ is allowed to grab. As every process belongs to a user, you need to bind the setting to a Linux user (here, the `vernemq` user).
To do this, edit `/etc/security/limits.conf`, for which you'll need superuser access. If you installed VerneMQ from a binary package, add lines for the `vernemq` user, substituting your desired hard and soft limits:

```text
vernemq soft nofile 65536
vernemq hard nofile 262144
```

On Ubuntu, if you’re always relying on the init scripts to start VerneMQ, you can create the file /etc/default/vernemq and specify a manual limit:

```text
ulimit -n 262144
```

This file is automatically sourced from the init script, and the VerneMQ process started by it will properly inherit this setting. As init scripts are always run as the root user, there’s no need to specifically set limits in `/etc/security/limits.conf` if you’re solely relying on init scripts.

On CentOS/RedHat systems, make sure to set a proper limit for the user you’re usually logging in with to do any kind of work on the machine, including managing VerneMQ. On CentOS, `sudo` properly inherits the values from the executing user.

### Linux and Systemd service files

Newer VerneMQ packages use a systemd service file. You can adapt the `LimitNOFILE` setting in the `vernemq.service` file to the value you need. It is set to `infinity` by default already, so you only need to adapt it in case you want a lower value. The reason we need to enforce the setting is that systemd doesn't automatically take over the `nofile` settings from the OS.

```text
LimitNOFILE=infinity
```

## Enable PAM-Based Limits for Debian & Ubuntu

It can be helpful to enable PAM user limits so that non-root users, such as the `vernemq` user, may specify a higher value for maximum open files. For example, follow these steps to enable PAM user limits and set the soft and hard values **for all users of the system** to allow for up to 65536 open files.

Edit `/etc/pam.d/common-session` and append the following line:

```text
session    required   pam_limits.so
```

If `/etc/pam.d/common-session-noninteractive` exists, append the same line as above.

Save and close the file.

Edit `/etc/security/limits.conf` and append the following lines to the file:

```text
*               soft     nofile          65536
*               hard     nofile          262144
```

1. Save and close the file.
2. \(optional\) If you will be accessing the VerneMQ nodes via secure shell \(ssh\), you should also edit `/etc/ssh/sshd_config` and uncomment the following line:

```text
#UseLogin no
```

and set its value to `yes` as shown here:

```text
UseLogin yes
```

1. Restart the machine so that the limits to take effect and verify

   that the new limits are set with the following command:

```text
ulimit -a
```

## Enable PAM-Based Limits for CentOS and Red Hat

1. Edit `/etc/security/limits.conf` and append the following lines to

   the file:

```text
*               soft     nofile          65536
*               hard     nofile          262144
```

1. Save and close the file.
2. Restart the machine so that the limits to take effect and verify that the new limits are set with the following command:

```text
ulimit -a
```

{% hint style="info" %}
In the above examples, the open files limit is raised for all users of the system. If you prefer, the limit can be specified for the `vernemq` user only by substituting the two asterisks \(\*\) in the examples with `vernemq`.
{% endhint %}

## Solaris

In Solaris 8, there is a default limit of 1024 file descriptors per process. In Solaris 9, the default limit was raised to 65536. To increase the per-process limit on Solaris, add the following line to `/etc/system`:

```text
set rlim_fd_max=262144
```

Reference:

## Mac OS X

To check the current limits on your Mac OS X system, run:

```text
launchctl limit maxfiles
```

The last two columns are the soft and hard limits, respectively.

To adjust the maximum open file limits in OS X 10.7 \(Lion\) or newer, edit `/etc/launchd.conf` and increase the limits for both values as appropriate.

For example, to set the soft limit to 16384 files, and the hard limit to 32768 files, perform the following steps:

1. Verify current limits:

   > ```text
   > launchctl limit
   > ```
   >
   > The response output should look something like this:
   >
   > ```text
   > cpu         unlimited      unlimited
   > filesize    unlimited      unlimited
   > data        unlimited      unlimited
   > stack       8388608        67104768
   > core        0              unlimited
   > rss         unlimited      unlimited
   > memlock     unlimited      unlimited
   > maxproc     709            1064
   > maxfiles    10240          10240
   > ```

2. Edit \(or create\) `/etc/launchd.conf` and increase the limits. Add lines that look like the following \(using values appropriate to your environment\):

   > ```text
   > limit maxfiles 16384 32768
   > ```

3. Save the file, and restart the system for the new limits to take effect. After restarting, verify the new limits with the launchctl limit command:

   > ```text
   > launchctl limit
   > ```
   >
   > The response output should look something like this:
   >
   > ```text
   > cpu         unlimited      unlimited
   > filesize    unlimited      unlimited
   > data        unlimited      unlimited
   > stack       8388608        67104768
   > core        0              unlimited
   > rss         unlimited      unlimited
   > memlock     unlimited      unlimited
   > maxproc     709            1064
   > maxfiles    16384          32768
   > ```

**Attributions**

This work, "Open File Limits", is a derivative of Open File Limits by Riak, used under Creative Commons Attribution 3.0 Unported License. "Open File Limits" is licensed under Creative Commons Attribution 3.0 Unported License by Erlio GmbH.

