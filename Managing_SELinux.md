# Understanding the Need for SELinux
- Linux security is built on UNIX security
- UNIX security consists of different solutions that were never developed with current IT security needs in mind
- Most of these solutions focus on a part of the operating system
- SELinux provides a complete and mandatory security solution
- The prinicple is that if it isn`t specifically allowed, it will be denied
- As a result, "unknown" services will always need additional configuration to enable them in an environment where SELinux is enabled

# Managing SELinux Modes
enabled -> enforcing, permissive; setenforce 

kernel level (reboot required for changes)

disabled

- `getenforce` will show the current state
- `setenforce` toggles between enforcing and permissive
- Edit the /etc/sysconfig/selinux to manage the default state of SELinux
- Never set to disabled if this is meant as a temporary measure only!

```
getenforce
setenforce permissive
getenforce
setenforce disabled
vim /etc/sysconfig/selinux
reboot
getenforce
setenforce enforcing
vim /etc/sysconfig/selinux
reboot
getenforce
```

# Understanding SELinux context labels and booleans
