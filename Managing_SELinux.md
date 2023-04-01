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
- Every object is labeled with a context label
- - user: user specific context
- - role: role specific context
- - type: flags which type of operation is allowed on this object
- Many commands support a `-Z` option to show current context information
- Context tpyes are used in the rules in the policy to define which source object has access to which target object
- A Boolean is an on/off switch
- Use it to enable or disable specific categories of functionality altogether

```
ps aux
ps auxZ
ps auxZ | grep ssh
ls -lZd /etc/ssh/
ls -lZ /etc/ssh/
ps auxZ | grep httpd
ls -Z /var/www
ls -lZ /home/user
getsebool -a
getsebool -a | grep http
setsebool -P httpd_enable_homedirs on
```

# Using file context labels
- Use `semanage fcontext` to set the file context label
- - This will write the context to the SELinux Policy
- To enforce the policy setting on the file system, use `restorecon`
- Alternatively, use `touch /.autorelabel` to relabel all files to the context that is specified in the policy

```
mkdir /web
cd /web
echo "welcome to the webserver directory" > index.html
sed -i 's/DocumentRoot.*/DocumentRoot "\/web/' /etc/httpd/conf/httpd.conf
sed -i 's/<Directory.*/<Directory> "\/web/' /etc/httpd/conf/httpd.conf
systemctl restart httpd
systemctl status httpd
curl http://localhost
setenforce 0
getenforce
!curl
setenforce 1
man semanage-fcontext
semanage fcontext -a -t httpd_sys_content_t "/web(/.*)?"
!curl
ls -ldZ /web
restorecon -Rv /web
!curl
ps Zaux | grep httpd
ls -Zd /web
```

# Analyzing SELinux Log Messages
- SELinux uses `auditd` to write log messages to the audit log
- Messages in the audit log may be hard to interpret
- Ensure that `sealert` is available, it interprets messages from the audit log, applies SELinux AI, and writes meaningful messages to /var/log/messages
- Run the `sealert` command, including the UUID message to get advice on how to troubleshoot specific issues

```
grep AVC /var/log/audit/audit.log
journalctl | grep sealert
sealert -l 143234343433 | less
```

# Resetting the Root password and SELinux
```
reboot
crtl + e -> rd.break
mount
mount -o remount,rw /sysroot
chroot /sysroot
passwd
ls -Z /etc/shadow
load_policy -i 
ls -Z /etc/shadow
restorecon -v /etc/shadow
touch /.autorelabel
reboot
```

# Troubleshooting SELinux
```
systemctl start httpd
systemctl status httpd
getenforce
setenforce 0
systemctl start httpd
systemctl status httpd
grep sealert /var/log/messages
semanage port -a -t http_port_t -p tcp 83
setenforce 1
systemctl stop httpd; systemctl start httpd
systemctl status httpd
getenforce

restorecon -Rv /etc
cd
cp /etc/hosts .
ls -Z hosts
ls -Z /etc/hosts
rm -f /etc/hosts
mv hosts /etc/
ls -Z /etc/hosts
restorecon -Rv /etc
```


