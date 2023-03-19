# Understanding Troubleshooting Modes

    services    -> rescue.target
    baseos      -> systemd.unit = emergency.target
    systemd     ->  init = /bin/bash
    Kernel, initramfs  -> Rd.break
    GRUB  -> Grub menu: kernel arguments
UEFI    BIOS    -> Boot-disk
POST  (power on self test)

# Changing the root password
- Enter grub menu while booting
- Find the line that loads the Linux kernel and add `rd.break` to the end of the line
- `mount -o remount,rw /sysroot`
- `chroot /sysroot`
- `echo secret | passwd --stdin root`
- `touch /.autorelabel`
- CRTL-D
- CTRL-D

```
reboot
# enter grub menu and enter rd.break
mount
mount -o remount,rw /sysroot
mount
cd sysroot/
ls
chroot /sysroot
echo password | passwd --stdin root
touch /.autorelabel
#CRTL-D
#CRTL-D
```

# Troubleshooting filesystem issues
- real corruption does occur, but not often, and is automatically fixed
- Problems occur when making typo's in /etc/fstab
- To fix: if necessary, remount filesystem in read/write state and edit /etc/fstab
- Fragmentation can be an issue, different tools exist to fix
- - `xfs_fsr` is the XFS File System Reorganizer, it optimizes XFS file systems
- - `e4defrag` can be used to defragment Ext4

```
cat >> /etc/fstab << EOF
/dev/sda1       /nowhere        ext4        defaults    0 0 EOF
mount -a
mkdir /nowhere
mount -a
reboot
```

# Troubleshooting Networking issues
- wrong subnet mask
- wrong router
- DNS not working

```
ping centos.org
ip a
ip a d 192.168.4.235/32 dev ens33
ip a a dev ens33 192.168.4.235/24
ip a
!pi
ip route show
ip route add default via 192.168.4.2
ping centos.org
nmtui
dhclient
ip a
```

# Troubleshooting Performance issues
- Troubleshooting performance is an art on its own
- Focus on the four key areas of performance
- - memory
- - cpu load
- - disk load
- - network

```
top
renice
kill
```

# Troubleshooting Software issues
- Dependency problems in RPM's
- - should not occur when using repositories
- Library problems
- - run `ldconfig` to update the library cache

```
ls
rpm -ivh nmap-7.6.rpm
yum install ./nmap-7.6.rpm
yum install nmap
```

# Troubleshooting Memory shortage
`top`


