---
description: A guide that shows how to change the open file limtits
---

# Change Open File Limits

VerneMQ can consume a large number of open file handles when thousands of clients are connected as every connection requires at least one file handle.

Most operating systems can change the open-files limit using the `ulimit -n` command. Example:

```text
ulimit -n 65536
```

However, this only changes the limit for the _**current shell session**_. Changing the limit on a system-wide, permanent basis varies more between systems.

## Linux

On most Linux distributions, the total limit for open files is controlled by `sysctl`.

```text
sysctl fs.file-max
fs.file-max = 50384
```

As seen above, it is generally set high enough for VerneMQ. If you have other things running on the system, you might want to consult the [sysctl manpage](http://linux.die.net/man/8/sysctl) manpage for how to change that setting. However, what most needs to be changed is the per-user open files limit. This requires editing `/etc/security/limits.conf`, for which you'll need superuser access. If you installed VerneMQ from a binary package, add lines for the `vernemq` user like so, substituting your desired hard and soft limits:

```text
vernemq soft nofile 4096
vernemq hard nofile 65536
```

On Ubuntu, if you’re always relying on the init scripts to start VerneMQ, you can create the file /etc/default/vernemq and specify a manual limit like so:

```text
ulimit -n 65536
```

This file is automatically sourced from the init script, and the VerneMQ process started by it will properly inherit this setting. As init scripts are always run as the root user, there’s no need to specifically set limits in `/etc/security/limits.conf` if you’re solely relying on init scripts.

On CentOS/RedHat systems, make sure to set a proper limit for the user you’re usually logging in with to do any kind of work on the machine, including managing VerneMQ. On CentOS, `sudo` properly inherits the values from the executing user.

## Systemd
Systemd allows you to set the open file limit. The LimitNOFILE parameter defines the maximum number of file descriptors that a service or system unit can open. In the past, "infinite" was often chosen, which actually means an OS/systemD dependent maximum number. However, in recent versions of systemd like RHEL 9, CentOS Stream 9, and others, the default value is set to around a billion, significantly higher than necessary and the defaults used in older distributions. It is advisable to set a reasonable default value for LimitNOFILE based on the specific use case. Please consult https://access.redhat.com/solutions/1479623 for more information (RHEL9).

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
*               hard     nofile          65536
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
*               hard     nofile          65536
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
set rlim_fd_max=65536
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

