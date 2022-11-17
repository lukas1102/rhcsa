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
