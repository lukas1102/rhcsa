# Understanding RHEL 8 Logging Options
Rsyslogd -> /var/log

Systemd
Systemd-journald (not persistent)&nbsp;&nbsp;&nbsp;&nbsp;-> /dev/log -> Rsyslog
&nbsp;&nbsp;&nbsp;&nbsp;journalctl&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-> /var/log/journal

# Configuring Rsyslog Logging
- Rsyslog needs the rsyslogd service to be running
- The main configuration file is /etc/rsyslog.conf
- Snap-in files can be placed in /etc/rsyslog.d/
- Each logger line contains three items
- - facility: the specific facility that the log is created for
- - severity: the severity from which should be logged
- - destination: the file or other destination the log should be written to
- Log files normally are in /var/log
- Use the `logger` command to write messages to rsyslog manually
- rsyslogd is amust be backwards compatible with the archaic syslog service
- in syslog, a fixed number of facilities was defined, like kern, authpriv, cron and more
- to work with services that don´t have their own facility local{0..7} can be used
- Because of the lack of facilities, some services take care of their own logging and do not use rsyslog

# Working with systemd-journald
- systemd-journald is the log service that is part of systemd
- it integrates well with systemctl status <init> output
- alternatively, the journalctl command can be used to read log entries in journal
- Messages are logged also to rsyslogd, using the rsyslogd imjournal module
- To make the journal persistent, use `mkdir /var/log/journal`

```
mkdir /var/log/journal
vim /etc/systemd/journald.conf
journalctl
journalctl UNIT=sshd
systemctl status httpd
```

# Preserving the Systemd Journal
- The journal is written to /run/log/journal, which is automatically cleared on system reboot
- Eidt /etc/systemd/journald.conf to make the journal persistent across reboots
- Set the Storage parameter in this file to the appropriate value
- - persistent will store the journal in the /var/log/journal directory. This directory will be created if it doesn't exist
- - volatile stores the journal only in /run/log/journal
- - auto will store the journal in /var/log/journal if that directory exists, and in /run/log/journal if no /var/log/journal exists

## Understanding systemd journal log rotation
- Built-in log rotation for the journal runs monthly
- The journal cannot grow beyond 10% of the size of the file system it is on
- The journal will also make sure at least 15% of its file system will remain available as free space
- These settings can be changed through /etc/systemd/journald.conf

```
systemctl status systemd-journald
systemctl status systemd-journald -l
vim /etc/systemd/journald.conf
mkdir /var/log/journal
systemctl restart systemd-journald
systemctl status systemd-journald
```

# Configuring Logrotation
- Logrotate is started through cron.daily to ensure that log files don't grow too big
- Main configuration is in /etc/logrotate.conf, snap-in files can be provided through /etc/logrotate.d/
