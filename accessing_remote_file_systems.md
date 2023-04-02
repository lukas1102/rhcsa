# Configuring a Base NFS Server
- Run the nfs-server service
- Create a directory you want to share: /data
- Edit /etc/exports to contain the following line
- - /data *(rw,no_root_squash)
- Open three firewalld services: nfs mountd rpc-bind

```
systemctl status nfs-server
mkdir /data
cat > /etc/exports << EOF
/data   *(rw,no_root_squash)
EOF
systemctl enable --now nfs-server
systemct status nfs-server
firewall-cmd --add-service nfs
firewall-cmd --add-service mountd
firewall-cmd --add-service rpc-bind
firewall-cmd --add-service nfs --permanent
firewall-cmd --add-service mountd --permanent
firewall-cmd --add-service rpc-bind --permanent
firewall-cmd --list-all
```

# Mounting NFS Shares
- Use `showmount -e <nfs-server>` to show exports
- Use `mount nfsserver:/share /mnt` to mount
- While mounting through /etc/fstab, include the `_netdev` mount options

```
showmount -e 192.168.4.210
mount 192.168.4.210:/data /mnt
mount
umount /mnt
mkdir /nfs
cat >> /etc/fstab << EOF
192.168.4.210:/data         /nfs        nfs     _netdev         0  0
EOF
mount -a
mount
```

# Configuring a Base Samba Server
- Install the Samba server package
- Create a directory to share
- Create a local Linux user
- Set Linux Permissions
- Use `smbpasswd -a` ot add a Samba user account
- Enable the share in /etc/samba/smb.conf
- Use `systemctl start smb` to start the service
- Use `firewall-cmd --add-service samba --permanent; firewall-cmd --reload` to open the firewall

```
yum install samba
mkdir /samba
useradd samba
chown samba /samba
chmod 770 /samba
smbpasswd -a samba
cat >> /etc/samba/smb.conf << EOF
[samba]
        comment = samba share
        path = /samba
        write list = samba
EOF
systemctl enable --now smb
systemctl status smb
firewall-cmd --add-service samba --permanent
firewall-cmd --reload
firewall-cmd --list-all
```

# Mounting samba shares
- Install the cifs-utils and samba-client RPm packages
- Use `smbclient -L //sambahost` to discover shares
- Use `mount -o username=sambauser //sambaserver/share /somewhere` to mount the share
- Make mount persisten through /etc/fstab, using the _netdev, username= and password= mount options

```
yum groups list
yum groups list --hidden | less
yum groups install "Network File System Client"
smbclient -L //192.168.4.210
mount -o username=samba //192.168.4.210/samba /mnt
mount
cat >> /etc/fstab << EOF
//192.168.4.210/samba       /samba      cifs    _netdev,username=samba,password=password        0  0
EOF
mkdir /samba
mount -a
mount
reboot
mount
```

# Understanding Automount
- In `/etc/auto.master` you'll identify the directory that automount should manage, and the file that is used for additional mount information
- - `/data  /etc/auto.data`
- In `/etc/auto.data` you'll identify the subdirectory on which to mount, and what to mount exactly
- - `files  -rw     nfsserver:/data/files`
- Ensure the autofs service is started
- - `systemctl enable --now autofs`

```
showmount -e 192.168.4.210
yum search autofs
yum install autofs
ls /
systemctl enable --now autofs
ls /
vim /etc/auto.master
vim /etc/auto.misc
cat >> /etc/auto.master << EOF
/files /etc/auto.files
EOF
cat > /etc/auto.files << EOF
data    -rw     192.168.4.210:/data
EOF
systemctl restart autofs
ls /
cd /files
ls -a
cd data
pwd
mount | grep data
```

# Configuring Automount for Home Directories
```
su - ldapuser1
exit
ssh 192.168.4.210
echo "/home/ldap    *(rw)" > /etc/exports
systemctl restart nfs-server
exit
showmount -e 192.168.4.210
echo "/home/ldap    /etc/auto.ldap" >> /etc/auto.master
cat > /etc/auto.ldap << EOF
*       -rw     192.168.4.210:/home/ldap/&
EOF
systemctl restart autofs
su - ldapuser1
pwd
touch hello
ls -l
```
