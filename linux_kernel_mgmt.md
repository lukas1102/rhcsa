# Understanding the Linux Kernel
shell (user space)
syscall

kernel
- InitRamfs (initial drivers)
- Systemd-Udevd  (hot plug drives)
- modprobe (manually load drivers)

drivers-modules
hardware

#  Working with Kernel Modules
- Linux drivers are implemented as kernel modules
- Most kernel modules are loaded automatically through initramfs or systemd-udevd
- Use `modprobe` to manually load kernel modules
- use `lsmod` to list currently loaded kernel modules

```
lsmod | less
lsmod | grep vfat
modprob vfat
lsmod | grep vfat
modprobe -r vfat
```

# Using modprobe
- use `modprobe` to load a kernel module and all its dependencies
- use `modprobe -r` to unload
- `modinfo` can show module parameters
- to load, specify kernel module parameters, edit /etc/modprobe.conf or the files in /etc/modprobe.d

```
lsmod
modinfo e1000
cd /etc/modprobe.d
ls
vim kvm.conf
modprobe kvm
```

# Using /proc to tune kernel behavior
## understanding kernel parameters
- /proc is a file system that provides access to kernel information
- - PID directories
- - Status files
- - tuneables in /proc/sys
- use `echo` to write a value to any file in /proc/sys to change kernel performance parameters
- Write the parameters to /etc/sysctl.conf to make them persistent
- use `sysctl -a` to show a list of all current settings

```
mount | grep proc
cd /proc
ls
cat meminfo
cd sys
ls
cd net
ls
cd ipv4
ls
cat ip_forward
echo 1 > ip_forward
pwd
sysctl -a | grep forward
cat ip_forward
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -a | wc
cd /usr/lib/sysctl.d/
ls
```

# Updating the kernel
- Linux kernel are not technically updated, a new kernel is installed beside the old kernel
- This allows administrators to boot the old kernel in case anything goes wrong
- use either `yum update kernel` or `yum install kernel` to update the kernel
