# understanding Systemd units
systemd is the manager of everything after the start of the Linux kernel
- managed items are called units
- different unit types are available
- - services
- - mounts
- - timers
- - and many more
- systemctl is the management interface to work with systemd
- managing services is the most important systemd related task for an administrator
```
systemctl -t help
systemctl list-unit-files
systemctl list-units
```
# managing systemd services
- system administrators must be able to manage the state of modules
- disabled/enabled determine if a module should be automatically started while booting
- start/stop is managing runtime state of a service
```
yum install vsftpd
systemctl status vsftpd
systemctl start vsftpd
systemctl status vsftpd
systemctl enable vsftpd
```
# modifying systemd service configuration
- default system-provided systemd unit files are in /usr/lib/systemd/system
- custom unit files are in /etc/systemd/system
- run-time automatically generated unit files are in /run/systemd
- while modifying a unit file, do NOT edit file in /usr/lib/systemd/system, but create a custom file in /etc/systemd/system that is used as an overlay file
- Better: use `systemctl edit unit.service` to edit unit files
- use `systemctl show` to show available parameters
- using `systemctl-reload` may be required

```
systemctl enable --now vsftpd
systemctl cat vsftpd.service
systemctl show vsftpd.service
systemctl edit vsftpd.service
systemctl daemon-reload
systemctl restart vsftpd.service
systemctl status vsftpd.service
killall vsftpd
systemctl status vsftpd.service
```