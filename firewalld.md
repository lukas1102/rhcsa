# Understanding RHEL 8 Firewalling

firewalld
  |
nftables
  |
kernel
netfilter

# Understanding Firewalld components
Firewalld is using different components to make firewalling easier
- Service: the main component, contains one or more ports as well as optional kernel modules that should be loaded
- Zone: a default configuration to which network cards can be assigned to apply specific settings
- Ports: optional elements to allow access to specific ports
- Additional components are available as well, but not frequently used in a base firewall configuration

```
firewall-cmd --list-all
```

# Configuring a Firewall with firewall-cmd
- The `firewall-cmd` command is used to write firewall configuration
- Use the option `--permanent` to write to persistent (but not to runtime)
- Without `--permanent` the rule is written to runtime (but not to persistent)

```
firewall-cmd --get-services
firewall-cmd --add-service ftp
firewall-cmd --list-all
firewall-cmd --reload
firewall-cmd --list-all
firewall-cmd --add-service ftp --permanent
firewall-cmd --list-all
firewall-cmd --add-service ftp
```

# Using firewall-config

```
yum search firewall-config
yum install firewall-config
firewall-config
^Z
bg
firewall-cmd --list-all
fg

firewall-cmd --list-all
firewall-cmd --get-services
firewall-cmd --add-service http
firewall-cmd --add-service https
firewall-cmd --add-service http --permanent
firewall-cmd --add-service https --permanent
systemctl restart firewalld
firewall-cmd --list-all
```
