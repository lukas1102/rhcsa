# Understanding Disk layout
/dev/nvmeam
/dev/sda
 - sda1
 - sda2
/dev/vda

BIOS: 
MBR (Master Boot Record)
4 partitions!
logical partition
extended 


UEFI: 
GPT (GIUD Partition Table)
128 partitions!

```
lsblk
parted /dev/nvme0n1
    print
    quit
cd /dev
ls -l nvm*
ls -l sd*
cat /proc/partitions
```

# Understanding Linux Storage options
- Partitions: the classical solution, use in all cases
- - Use to allocate dedicated storage to specific types of data
- LVM Logical Volumes
- - Used at default installation of RHEL
- - Adds flexibility to storage (resize, snapshots and more)
- Stratis
- - Next generation Volume Managing Filesystem that uses thin provisioning by default
- - Implemented in user space, which makes API access possible
- Virtual Data Optimizer
- - Focused on storing files ini the most efficient way
- - Manages deduplicated and compressed storage pools

# Understanding GPT and MBR partitions
- Master Boot Record (MBR) is part of the 1981 PC specification
- - 512 bytes to store boot information
- - 64 bytes to store partitions
- - Place for 4 partitions only with a max size of 2 TiB
- - To use more partitions, extended and logical partitions must be used
- GUID Partition Table (GPT) is a newer partition table (2010)
- - More space to store partitions
- - Used to overcome MBR limitations
- - 128 partitions max

# Creating Partitions with parted
- While creating a partition, you do NOT automatically create a file system
- The parted file system attribute only writes some unimportant file system metadata
- In RHEL 8, `parted` is the default utility
- Alternatively, use `fdisk` to work with MBR and `gdisk` to use GUID partitions

## Procedure overview 
- `parted /dev/sdb`
- `print` will show if there is a current partition table
- `mklabel msdos|gpt`
- `mkpart part-type name fs-type start end`
- - part type: applies to MBR only and sets primary, logical, or extended partition
- - name: arbitrary name, required for GPT
- - fs-type: does NOT modify the filesystem, but sets some irrelevant file system dependent metadata
- - start end: specify start and end, counting from the beginning of the disk

- - for instance: `mkpart primary 1024MiB 2048MiB`
- alternatively, use `mkpart` in interactive mode
- `print` to verify creation of the new partition
- `quit` to exit the parted shell
- `udevadm settle` to ensure that the new partition device is created
- `cat /proc/partitions` to verify the creation of the partition

# Creating MBR Partitions with fdisk
```
lsblk
fdisk /dev/nvme0n3
m
n
p
1
+1G
p
w
q
partprobe
```

# Understanding file system differences
- XFS is the default file system
- - fast and scalable
- - uses CoW (copy on write) to guarantee data integrity
- - Size can be increased, not decreased
- Ext4 was default in RHEL 6 and is still used
- - backwards compatible to Ext2
- - uses journal to guarantee data integrity (journal keeps track of what file is currently written to)
- - size can be increased and decreased
- Other file systems are available but less common

# Making and Mounting File Systems
- `mkfs.xfs` creates an XFS file system
- `mkfs.ext4` creates an Ext4 file system
- Use `mkfs.[Tab][Tab]` to show a list of available file systems
- Do NOT use `mkfs` as it will create an Ext2 file system!
- After making the file system, you can mount it in runtime using the `mount` command
- Use `umount` before disconnecting a device

```
lsblk
mkfs.xfs /dev/nvme0n3p1
mount /dev/nvme0n3p1 /mnt
mount
mount | grep '^/'
cd /mnt
ls /mnt
umount /dev/nvme0n3p1
lsof /mnt
cd
lsof /mnt
umount /mnt
mkfs.ext4 --help
mkfs.ext4 /dev/nvme0n3p2
mkfs.vfat
```

# Mounting Partitions through /etc/fstab
- /etc/fstab is the main configuration file to persistently mount partitions
- /etc/fstab content is used to generate systemd mounts by the systemd-fstab-generator utility
- to update systemd, make sure to use `systemctl damon-reload` after editing /etc/fstab

```
cat >> /etc/fstab << EOF
/dev/nvme0n3p1      /xfs        xfs     defaults    0 0
EOF
systemctl daemon-reload
mkdir /xfs 
mount -a
mount
```

# Managing Persistent Naming Attributes
- In datacenter environments, block device names may change. Different solutions exist for persistent naming
- UUID: a UUID (universal unique ID) is automatically generated for each device that contains a file system or anything similar (like swap devices)
- Label: while creating the file system, the option -L can be used to set an arbitrary name that can be used for mounting the file system
- Unique device names are created in /dev/disk


```
lsblk
fdisk /dev/nvme0n3
p
n
+2G
w
mkfs.xfs /dev/nvme0n3p5
mkfs.ext4 /dev/nvme0n3p6
mkdir /books /articles
cat >> /etc/fstab << EOF
/dev/nvme0n3p5      /books        xfs     defaults    0 0
/dev/nvme0n3p6      /articles        ext4     defaults    0 0
EOF
mount -a
fdisk /dev/nvme0n3
d
5
w
sed -i 's/\/dev\/nvme0n3p5/#\/dev\/nvme0n3p5/' /etc/fstab
reboot
lsblk
blkid
tune2fs --help
xfs_admin --help
tune2fs -L articles /dev/nvme0n3p5
blkid
sed -i 's/*\/dev\/nvme0n3p5*/LABEL=articles     \/articles  ext4    defaults    0 0'  /etc/fstab
mount | grep artic
mount -a | grep artic
cd /dev/disk
ls by-path/
```

# Managing Systemd mounts
- /etc/fstab mounts already are systemd mounts
- Mounts can be created using systemd .mount file
- Using .mount files allows you to be more specific in defining dependencies
- Use `systemctl cat tmp.mount` for an example

```
systemctl cat tmp.mount
mount -a | grep tmp
systemctl status tmp.mount
systemctl enable --now tmp.mount
systemctl status tmp.mount
mount | grep tmp
cp /usr/lib/systemd/system/tmp.mount /etc/systemd/system/data-articles.mount
vim /etc/systemd/system/data-articles.mount
systemctl daemon-reload
systemctl status data-articles.mount
umount /articles
systemctl status data-articles.mount
systemctl enable --now data-articles.mount
systemctl status data-articles.mount
```

# Managing XFS File Systems
- The `xfsdump` utility can be used for creating backups of XFS formatted devices and considers specific XFS attributes
- - xfsdump only works on a complete XFS device
- - xfsdump can make full backups (-l 0) or different levels of incremental backups
- - `xfsdump -l 0 -f /backupfiles/data.xfsdump /data` creates a full backup of the contents of the /data directory
- The xfsrestore command is used to restore a backup that was made with xfsdump
- - `xfsrestore -f /backupfiles/data.xfsdump /data`
- The xfsrepair command can be manually started to repair broken XFS file systems

# Creating a Swap Partition
- Swap is RAM that is emulated on disk
- All Linux system shold have at least some swap
- - the amount of swap depends on the use of the server
- Swap can be created on any block device, including swap files
- While creating swap with `parted`, set file system to linux-swap
- After creating the swap partition, use `mkswap` to create the swap FS
- Activate using `swapon`

```
parted /dev/nvme0n2
print
mkpart
swap
linux-swap
1GiB
2GiB
print
quit
lsblk
mkswap /dev/nvme0n2p2
free -m
swapon /dev/nvme0n2p2
cat >> /etc/fstab << EOF
/dev/nvme0n2p2      swap        swap        defaults    0 0 EOF
```

# Understanding LVM, Stratis and VDO
## Understanding Advanced Storage
- LVM Logical Volumes
- - Used during default installation of RHEL
- - Adds flexibility to storage (resize, snapshots, and more)
- Stratis
- - Next generation Volume Managing Filesystem that uses thin provisioning by default
- - Implemented in user space, which makes API access possible
- Virtual Data Optimizer
- - Focused on storing files in the most efficient way
- - Manages deduplicated and compressed storage pools

## Understanding LVM setup

   ext4    xfs
    LV      LV
    |       |
volume group (abstraction of all the available storage on the system)
    |
storge disks, partitions  (physical volumes pv)

## Creating an LVM Logical Volume
- create a partition, from `parted` use `set n lvm on`
- use `pvcreate /dev/sdb1` to create the physical volume
- use `vgcreate vgdata /dev/sdb1` to create the volume group
- use `lvcreate -n lvdata -L 1G vgdata` to create the logical volume
- use `mkfs /dev/vgdata/lvdata` to create a file system
- put in /etc/fstab to mount it persistently

```
lsblk
pvcreate /dev/nvme0n2p3
pvs
vgcreate vgdata /dev/nvme0n2p3
pvs
vgs
lvcreate -n lvdata -L 812M vgdata
lvs
man lvcreate
mkfs.xfs /dev/vgdata/lvdata
mkdir -p /mounts/lvm1
echo "/dev/vgdata/lvdata        /mounts/lvm1     xfs     defaults    0 0" >> /etc/fstab
mount -a
mount
findmnt
```

## Understanding Device Mapper and LVM Device Names
- Device mapper is the system that the kernel uses to interface storage devices
- Device mapper generates meaningless names, like /dev/dm-0 and /dev/dm-1
- meaningful names are provided as symbolic links through /dev/mapper 
- - /dev/mapper/vgdata-lvdata
- alternatively, use the LVM generated symbolic links
- - /dev/vgdata/lvdata

```
mount
tail -n 1 /etc/fstab
ls -l /dev/vgdata/lvdata /dev/mapper/vgdata-lvdata
```

## Resizing LVM logical volumes
- use the `vgs` to verify availability in the volume group
- if required, use `vgextend` to add one or more PVs to the VG
- use `lvextend -ir -L +1G` to grow the LVM logical volume, including the file system it´s hosting
- - `e2resize` is an independent resize utility for ext file systems
- - `xfs_growfs` can be used to grow an XFS file system
- shrinking is not possible on XFS volumes

```
df -h
vgs
vgextend vgdata /dev/nvme0n2p4
vgs
lvextend -r -L +1020M /dev/vgdata/lvdata
df -h
```
## Understanding Stratis setup
- Stratis is a volume management file system and is Red Hats answer to Btrfs and ZFS
- - On top of Stratis a regular file system is needed: XFS
- It is built on top of any block device, including LVM devices
- It offers advanced features
- - Thin provisioning
- - snapshots
- - cache tier
- - programmatic API
- - monitoring and repair

- The Stratis Pool is created from one or more storage devices (blockdev)
- - Stratis creates a `/dev/stratis/my-pool` directory for each pool
- - This directory contains links to devices that represent the file systems in the pool
- - Block devices in a pool may not be thin provisioned
- The (XFS) file system is put in a volume on top of the pool and is an integrated part of it 
- - Each pool can contain one or more file systems
- - File systems are thin provisioned and do not have a fixed size
- - The thin volume which is an integrated part of the file system automatically grows as more data is added to the file system

## Creating Stratis Volumes
- `yum install stratis-cli stratisd`
- `systemctl enable --now stratisd`
- `stratis pool create mypool /dev/nvme0n2`
- - add new block devices later using `stratis blockdev add-data`
- - partitions are NOT supported
- - note that the block device must be at least 1GiB
- `stratis fs create mypool myfs1`
- - note this will create an XFS file system!
- `stratis fs list mypool` will show all file systems in the pool
- `mkdir /myfs1`
- `mount /dev/stratis/mypool/myfs1/myfs1`
- `stratis pool list`
- `stratis filesystem list`
- `stratis blockdev list mypool`
- `blkid to find the stratis volume UUID`
- mount by UUID in /etc/fstab

```
yum install -y stratis-cli stratisd
systemctl enable --now stratisd
stratis pool create mypool /dev/nvme0n2
stratis pool
stratis fs create mypool myfs1
stratis fs
mkdir /myfs1
mount | grep myfs1
mount /dev/stratis/mypool/myfs1/myfs1 /myfs1
mount
stratis pool list
stratis fs list
stratis blockdev list mypool
lsblk
blkid
cat >> /etc/fstab << EOF
UUID=34234234234234243      /myfs1   xfs    defaults    0 0  EOF
reboot
mount | grep myfs
```

## Extending a Stratis Pool
- Pools can be extended by adding additional block devices
- use `stratis pool add-data mypool /dev/nvme0n3` to add another block device

## Monitoring Stratis Volumes
- Standard Linux tools (df) don`t give accurate sizes as Stratis volumes are thin provisioned
- use `stratis blockdev` to show information about all block devices used for Stratis
- use `stratis pool` to show information about all pools
- - note thath Physical Used should not come too close to Physical size
- use `stratis filesystem` to monitor individual filesystems

## Understanding Stratis Snapshots
- A snapshot is an individual file system that can be mounted
- After creation, snapshots can be modified
- A snapshot and its origin are not linked: the snapshotted file system can live longer than the file system it was created from
- Each snapshot needs at least half a Gigabyte of backing storage for the XFS log

- `stratis fs snapshot mypool myfs1 myfs1-snapshot`
- - changes to the original FS will not be reflected in the snapshot
- - use `mount /stratis/mypool/my-fs-snapshot /mnt` to mount it
- Revert the original volume to the state in the snapshot
- - `unmount /myfs1`
- - `stratis fs destroy mypool myfs1`
- - `stratis fs snapshot mypool myfs1-snap myfs1`
- Note that this approach wouldn´t work on LVM

- `stratis filesystem destroy mypool mysnapshot` will delete a snapshot
- A similar procedure is used for destroying file systems: `stratis filesystem destroy mypool myfs`
- When there are no more file systems in a pool, use `stratis pool destroy mypool` to delete the pool

```
stratis blockdev
stratis pool
stratis filesystem
df -h | grep myfs
stratis filesystem snapshot mypool myfs1 myfs1-snapshot
stratis filesystem list
mount /dev/stratis/mypool/myfs1-snapshot /mnt
ls /mnt
cp /etc/a* /mnt
umount /myfs1
stratis filesystem destroy mypool myfs1
stratis filesystem list
stratis filesystem snapshot mypool myfs1-snapshot myfs1
mount -a
cd /myfs1/
ls
stratis filesystem destroy mypool myfs1-snapshot
umount /mnt
stratis filesystem destroy mypool myfs1-snapshot
```

## Understanding VDO
- VDO (Virtual Data optimizer) is used to optimize how data is stored on disk
- It is used as a separate volume manager on top of which file systems will be created
- Provides thin provisioned storage
- - use a logical size 10 times the physical size for VMs and containers
- - use a logical size 3 times the pyhsical size for object storage
- Used in Cloud/Containers environments
- It manages deduplicated and compressed storage in RHEL 8

## Configuring VDO Volumes
- Ensure that underlying block devices are > 4GiB
- `yum install vdo kmod-kvdo`
- `vdo create --name=vdo1 --device=/dev/nvme0n2p1 --vdoLogicalSize=1T`
- `mkfs.xfs -K /dev/mapper/vdo1`
- `udevadm settle` will wait for the system to register the new device name
- In /etc/fstab, include the `x-systemd.requires=vdo.service` and the `discard` mount options or use the systemd example file
- monitor using `vdostats --human-readable`

```
lsblk
yum install vdo kmod-kvdo
man vdo
vdo create --name=vdo1 --device=/dev/nvme0n2 --vdoLogicalSize=1T
modprobe kvdo
reboot
vdo create --name=vdo1 --device=/dev/nvme0n2 --vdoLogicalSize=1T
cd /usr/share/doc/vdo
ls
cd examples
ls
cd systemd
ls
mkdir /vdo1
cp VDO.mount.example /etc/systemd/system/vdo1.mount
sed -i 's/name.*/name = vdo1.mount/' /etc/systemd/system/vdo1.mount
sed -i 's/What.*/What = \/dev\/mapper\/vdo1/' /etc/systemd/system/vdo1.mount
sed -i 's/Where.*/Where = \/vdo1/' /etc/systemd/system/vdo1.mount

systemctl daemon-reload
systemctl enable --now vdo1.mount
systemctl status vdo1.mount
mkfs.xfs -K /dev/mapper/vdo1
systemctl enable --now vdo1.mount
systemctl status vdo1.mount
vdostats --human-readable
df -h
reboot
mount | grep vdo

```

## Understanding LUKS encryption volumes
/dev/sda1 -> cryptsetup luksFormat  -> cryptsetup luksOpen <secret> -> /dev/mapper/secret -> mkfs  -> mount

## Configuring LUKS encrypted volumes
- use `parted` to create a partition
- `cryptsetup luksFormat` will format the LUKS device
- `cryptsetup luksOpen` will open it and create a device mapper name
- mount the resulting device mapper device
- to automate the cyptsetup luksOpen, use /etc/crypttab
- to automate mounting the volume, use /etc/fstab

```
parted /dev/nvme0n2
print
mkpart
lsblk | less
cryptsetup luksFormat /dev/nvme0n2p5
cryptsetup luksOpen /dev/nvme0n2p5 secret
ls -l /dev/mapper
mkfs.xfs /dev/mapper/secret
mkdir /secret
cat >> /etc/fstab << EOF
/dev/mapper/secret      /secret   xfs    defaults    0 0  EOF
cat >> /etc/crypttab << EOF
secret /dev/nvme0n2p5 none  EOF
man crypttab
reboot
```